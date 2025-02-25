---
title: ElasticSearch Essential - 05 모니터링 하기
description: >-
  트러블 슈팅의 방법을 알아보고, 다양한 트러블 슈팅 사례를 분석해보자. 
author: ggong
date: 2024-12-20 20:58:00 +0900
categories: [Online Course, Elasticsearch]
tags: [online-course, elasticsearch, study]
pin: false
media_subpath: '/assets/img/_posts/online-course/elasticsearch'
references:
  - name: "ElasticSearch Essential(강진우)"
    url: "https://www.inflearn.com/course/elasticsearch-essential"
---

## 트러블 슈팅의 기본

트러블 슈팅
: 1.문제상황을 파악하고, 2.원인을 분석한 후 3.해결하는 과정을 말한다. 

### 문제상황 

아래는 문제상황을 파악하기 위한 흐름도의 예시다.
절대적인 방법은 아니지만, 좋은 방법이 될 수 있다.
중요한 것은 **어디에서 문제가 발생했는지, 어떤 문제가 발생했는지를 확인**하는 것이다.


**문제 상황 파악의 흐름**
1. 모니터링 알람 확인
: CPU USAGE가 80%, DISK의 USAGE가 80%인 경우 등

2. 클러스터 상태 확인
: 1번 과정에서 문제가 없었다면, cat API - health를 통해 클러스터의 상태를 확인한다. 

3. 노드에서 에러 로그 확인
: 2번 과정에서도 문제가 없었다면, 노드에서 발생한 로그중 에러로그는 없는지 확인한다.

4. 클라이언트에서 에러 로그 확인
: 3번 과정에서도 문제가 없었다면, 클라이언트에서의 로그까지 확인하며 문제의 상황을 파악한다.

### 원인 분석

원인 분석에서는 왜 문제가 발생했는지를 알아 내는 것이 주된 목표이다.

**원인 분석의 흐름**
1. 주요 지표 살펴보기
: 문제 상황에서 발견된 지표가 있다면, 해당 부분을 더 확인한다.

2. 에러 로그 분석
: 에러 로그를 기준으로 원인을 파악하고, 어떤 상황에서 문제가 발생헀는지 파악한다.

### 문제 해결

어떻게 해야 해결할 수 있는지 방법을 찾아본다.
원인 분석을 통해 확인한 원인을 제거하는 과정을 진행한다. 
이 내용은 이후 여러 다양한 사례를 통해 정리한다. 


## 클러스터의 상태 이상
> **상황:** 클러스터의 상태가 Green이 아니에요.
{: .prompt-warning }

### 문제상황 파악

문제 상황
: 클러스터의 상태가 Green이 아니다.
: yellow 혹은 red에 따른 영향도(=범위와 관리되고 있는 데이터의 중요도)를 정확하게 파악해야한다. 
  모든 인덱스가 레드인지, 데이터가 유실되어도 중요하지 않은 데이터 인지 등

> 클러스터 색에 따른 상태
> green
: 정상
: 프라이머리 샤드, 레플리카 샤드 모두 각 노드에 배치되어 동작하고 있는 상태
> 
> yellow
: 색인 성능은 이상 없고, **검색 성능에 이상이 있을 수 있음**
: 프라이머리 샤드는 정상적으로 동작하지만, 레플리카 샤드가 정상적으로 배치되지 않은 상태
> 
> red
: 색인 성능과 검색 **성능에 이상이 있을** 수 있고, **문서 유실이 발생할 수 있음**
: 일부 프라이머리 샤드와 레플리카 샤드가 정상적으로 배치되지 않은 상태
{:.prompt-info .ms-4}

### 원인 분석

원인
: 인덱스들 중에 누가 yellow 혹은 red 인가를 파악해야한다.

1. cat API 중 `indices`를 이용해 인덱스를 확인한다.
  ```shell
  curl -X GET "localhost:9200/_cat/indices" | grep -i red
  ```
  - 모든 인덱스가 나오기 때문에 `grep`과 조합해 `red`인 인덱스만 조회

2. 조회된 내용을 분석해 장애인 것인지 확인
  ```shell
    red   open  access_log_2022_11_17 ..... 5 1
    red   open  access_log_2022_11_19 ..... 5 1
    red   open  access_log_2022_11_20 ..... 5 1
  ```
  - 만약 오늘이 2022년 12월 1일이고, 3일이 지나면 백업을 해놓는다고 가정된 상태였다면 이 데이터들은 모두 백업데이터가 있는 인덱스이기 때문에 스냅샷에 있는것으로 리스토어 하면 그렇다면 문서의 유실이 없기 때문에 **장애가 아닌 상황**이다.
  - 만약 오늘이 2022년 11월 20일이라면, 장애의 상황이 맞기 때문에 **어떤 샤드에 문제가 문제가 생겼는지 파악을 해 장애의 범위와 얼마나 치명적인지를 파악**해야한다.

3. cat API 중 `shards`를 이용해 샤드를 확인한다.
  ```shell
    curl -X GET "localhost:9200/_cat/shards?h=idx,sh,pr,st,docs,unassigned.reason" | grep -iv started
  ```
  - 옵션을 추가해 헤더와 `unassigned.reason`이 함께 조회되며, `grep`과 조합해 상태가 `started`가 아닌 것만 조회

4. 조회된 내용을 분석해 어떤 인덱스에 어떤 샤드가 문제인지 확인
  ```shell
  index                         shard prirep  state       docs  unassigned.reason
  access_log_2022_11_17         1     prirep  UNASSIGNED  ..    NODE_LEFT
  access_log_2022_11_17         2     prirep  UNASSIGNED  ..    NODE_LEFT
  ```
  - `state`를 확인했을때, `UNASSIGNED`로 비정상인 것을 확인
  - 만약 10개의 샤드가 있는 상황이였다면, 조회된 결과에서 본 것과 같이 1번과 2번 샤드만 문제가 발생된 거라 0번, 3~9번 총 8개의 샤드는 정상적으로 동작한다.
  결과적으로 **현재 상황에서는 20%의 문서 유실이 발생할 수 있는 상황**이다. => **전체 다 유실된 것은 아니다**.


## 문서 색인 불가

> **상황**: Disk 사용량 초과로 인해 **클러스터에 갑자기 문서가 색인되지 않음** 혹은 **클라이언트에서 403 에러 발생**
{: .prompt-warning }


### 문제상황 파악

문제상황
: 클러스터에 갑자기 문서가 색인되지 않거나 클라이언트에서 403 에러가 발생하는 것이 문제


### 원인 분석

원인
: 색인이 계속 되는 상황이라면 하드디스크를 보호하는 설정인 `cluster.routing.allocation.disk.threshold_enabled`을 사용하는 것을 권고하는데,
  설정이 되어있는 상황에서 disk의 사용량이 너무 높아지면 더이상 색인을 하지 않는다.
  그렇기에 disk의 사용량이 어느정도 인지 확인해 이 설정과 관련된 것인지 확인한다.
: 403 에러가 발생하는 원인은, 저 설정으로 인해 모든 인덱스를 read-only로 변경하기 때문이다. 

> 하드디스크를 보호하는 설정
> `cluster.routing.allocation.disk.threshold_enabled`
: 보호장치를 사용할 것인지 아닌지를 설정, 무조건 사용하는 것을 권고함
>
> `cluster.routing.allocation.disk.watermark.low`
: 기본 값은 85% 이고, 이 값보다 높아지면 더이상 샤드를 배치하지 않음
>
> `cluster.routing.allocation.disk.watermark.high`
: 기본값은 90% 이고, 이 값보다 높아지면 샤드들은 다른 데이터 노드로 옮기기 시작
>
> `cluster.routing.allocation.disk.watermark.flood_stage`
: 기본값은 95% 이고, **이 값보다 높아지면 더이상 색인을 하지 않음**
{: .prompt-info .ms-4 }


###  해결 과정

과정 
1. 데이터 노드를 증설 하거나 불필요한 인덱스를 삭제해 디스크 공간을 확보한다.
2. read-only로 설정된 인덱스들을 명시적으로 풀어준다.
  ```shell
  PUT /*/settings
  {
     "index.blocks.read_only_allow_delete":null
  }
  ```

## 간헐적인 색인 누락


> **상황:** **간헐적**으로 색인 과정에서 일부 문서가 누락되요.
{: .prompt-warning }


### 문제상황 파악

문제 상황
: 간헐적으로 발생하는 문제 상황이라 쉽게 해결하기 어렵다.
: 노드에서 간헐적으로 `Rejected` 에러 발생시 이런 문제가 발생할 수 있다.

### 원인 분석

`Rejected` 에러
: elasticsearch에는 색인/검색 요청을 처리하는 스레드가 있고, 스레드가 모두 요청을 처리하고 있을 때를 대비해 큐도 존재하는데, 이 큐마저 모두 꽉 차있을 때 발생하는 에러를 말한다.  
: 알람지표나 스레드 관련 지표를 확인해 이 에러가 있었는지를 확인할 수 있다.

1. 알림 지표를 확인해 rejected 에러가 있었는지 확인한다.
2. 스레드 관련 지표를 확인해 이 에러가 있었는지 확인한다.

### 해결 과정

- 데이터 노드를 증설해 클러스터의 처리량을 늘려주는 방법
   - 이 방법이 더 좋지만, 샤드이 적절한 분배가 진행이 되어야 의미가 있고, 데이터 노드를 증설하기 어려울 수 있다.
- 큐를 증설해 데이터의 처리가 몰리는 경우의 대기하는 데이터의 수를 늘려주는 방법
   - 간헐적으로 많은 양의 요청의 인입으로 인한 경우라면, 큐 증설을 통해서도 대응할 수 있다.
  ```shell
  thread_pool.bulk.queue_size: [수 지정]
  ```


## 샤드 배치 불가

> **상황:** 샤드가 배치가 되지 않아요.
{: .prompt-warning }

### 문제상황 파악

문제 상황
: 샤드 배치가 되지 않으면, 프라이머리 샤드나 레플리카 샤드 또한 안만들어지며 yellow 나 red 가 되는 큰 이슈 상태이다.


### 원인 분석

> 노드 별로 샤드의 개수 제한 설정
> `cluster.max_shards_per_node`
: 각 노드별로 가질 수 있는 샤드의 수를 지정하는 설정이 있는데, 기본 값은 1000이다.
: 제한이 없다면, 리소스를 많이 사용하게 되어 리소스가 부족해지는 상황이 발생하기 때문이다.
{: .prompt-info }

### 해결 과정

- 데이터 노드 증설
  - 노드들이 가질 수 있는 샤드를 골고루 펼치고, 가질 수 있는 양을 늘리는 방법
- 인덱스 설정 변경
  - `number_of_shards`와 `number_of_replicas`를 줄여서 하나의 인덱스당 생성되는 샤드의 수를 줄이는 방법
- `cluster.max_shards_per_node` 변경
  - 값을 늘릴 수 있지만, 리소스가 많이 사용될 수 있기 때문에 권장되지 않는 방법


## 잦은 GC 발생

> **상황:** CMS GC 환경에서 너무 잦은 old GC가 발생해요.
{: .prompt-warning }



### 문제상황 파악

문제 상황
: GC 알고리즘 중 CMS 알고리즘을 사용하는 GC에서 old영역에 대한 GC가 너무 자주 일어나는 것을 의미한다.
: old영역에 GC가 발생하게 되면 [stop the world 현상](/posts/GC와_G1GC/#GC-stop-the-world){:.question target="_blank"}이 일어나게 되는데, 성능에 안좋은 영향을 주는 현상이다.

### 원인 분석

원인 분석
: 불필요하게 많은 객체들이 old 메모리 영역으로 이동하기 때문에 잦은 old GC가 발생하게 된다.
: elasticsearch의 원인이라기 보다는, jvm의 원인이라고 보는것이 좀 더 정확하다.
: JVM의 비주얼 GC를 붙여서 확인을 해야 정확하게 확인이 가능한데, 만약 survivor 영역이 작아 eden에서 바로 old 영역으로 넘어가는 것도 원인이 될 수 있다.


### 해결 과정

- survivor영역이 작았다면, 이 영역을 늘림
  - 튜닝을 하지 않았다면, young: old = 1:9 / eden : S0 : S1 = 8 : 1 : 1 이 기본적인 비율
  - 튜닝을 해서, young: old = 1:2 / eden : S0 : S1 = 6 : 1 : 1 로 변경
    - `XX:NewRatio=2`
    - `XX:SurvivorRatio=6`
- old GC가 발생되는 지점을 조절
  - old 영역에 75%가 차면 GC를 하는게 기본 값인데, 80%로 조절해 횟수를 늦춤 
    `XX:CMSSInitiatingOccupancyFraction=80`



## ThreadpoolWriteRejected 증가


> **상황:** 일부 문서만 색인이 되고, 일부 문서는 색인 시 에러가 발생해요
> AWS의 OpenSearch에서 발생했던 현상
{: .prompt-warning }

### 문제상황 파악

AWS의 OpenSearch에서 발생했던 이슈 사항이라, 여러가지 모니터링 도구가 있어 이를 통해 확인함
확인 결과 특정 노드에서만 리젝티드 에러가 발생하고 있는것을 알게됨



### 해결 과정

1. 이 경우에는 먼저 클러스터 전체가 계속해서 영향을 받기 때문에 클러스터에서 해당 노드를 제거해야하는 것이 우선이 되어야한다.
   ```shell
    PUT /_cluster/settings
   {
      "transient": {
          "cluster.routing.allocation.exclude._id": [해당 아이디]
      }  
   }
    ```
    - 서포트 티켓을 통해서 요청
2. 노드를 제외하고, 샤드를 다른 노드들로 이동 시킨 후 원인 분석을 하거나 다른 노드로 교체해야한다.
