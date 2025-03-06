---
title: Elasticsearch의 Query와 Filter
description: >-
  Elasticsearch에서 Query와 Filter가 무엇인지 알아보고, 둘의  차이점을 알아보자.
author: ggong
date: 2024-08-16 12:40:00 +0900
categories: [ Study, Data ]
tags: [ study, elasticsearch, topic, filter, search ]
pin: false
media_subpath: '/assets/img/_posts/online-course/elasticsearch'
references:
  - name: "Elasticsearch in Action"
    url: "http://www.acornpub.co.kr/book/elasticsearch-in-action-2e"
---

## Query

Query
: 문서의 **적합성(Relevance)**을 평가하여 **검색 점수(Score)**를 계산한다.
: **전문 검색(Full-text search)**을 수행하며, 검색어와 관련성이 높은 문서를 찾기 위해 사용된다.
: 검색 결과는 **`_score` 값에 따라 정렬된**다.
: 검색 점수 계산이 필요하기 때문에 **캐싱되지 않는다.**

```json  
{
  "query": {
    "match": {
      "title": "Elasticsearch filter query"
    }
  }
}
```

`title` 필드에서 `Elasticsearch filter query`와 **관련성이 높은 문서를 검색**되며, **`_score`값**이 높은 순서대로 정렬된다.

## Filter

Filter(필터)
: 문서가 **특정 조건을 만족하는지 여부만** 검사한다.
: 점수 계산 없이 **O(1) 연산처럼 동작하여** 빠르게 실행된다.
: 정확한 값 일치(match exact)나 범위 검색(range search)에 최적이다.
: [캐싱되기 때문에](#Filter-cache){: .question } 컴퓨팅 사이클을 일부 절약할 수 있다.

> **Filter의 값이 캐싱되는 이유**
> Filter 쿼리는 멱등성이 높아 성능 개선을 위해 자주 사용되는 Filter 조건을 캐시한다.
{: .prompt-question id="Filter-cache"}

```json
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "status": "Active"
          }
        },
        {
          "range": {
            "created_at": {
              "gte": "2024-01-01"
            }
          }
        }
      ]
    }
  }
}
```

`status`가 `Active`이고, `created_at`이 `2024-01-01` 이후인 문서만 필터링되며, `_score`값 없이 **조건을 만족하는 문서만** 반환된다.

## Query와 Filter

### 차이점

가장 큰 차이점은 [`_score`값](#Query와-Filter-score){: .question }의 유무이다.
Query는 `_score`값을 계산해 이를 기준으로 하고, Filter는 `_score`값이 아니라 조건 만족 여부만 확인한다.


> `_score`값이란?
: 검색된 문서들이 얼마나 검색어와 가깝고(연관성이 높고), 중요도가 높은지를 수치로 표현한 값이다.
: **TF-IDF(단어 빈도-역문서 빈도, Term Frequency-Inverse Document Frequency)**와 유사한 방식으로 동작한다.
{: .prompt-question id="Query와-Filter-score"}

### 함께 사용하기

bool로 query와 filter 분리
: [bool 쿼리](#함께-사용하기-bool){: .question }로 `_score`가 필요한 검색은 must, 필터는 filter를 이용해 검색한다.


> bool 쿼리 
: 여러 개의 조건을 조합하여 검색을 수행하는 방법으로, 각 조건을 논리 연산(AND, OR, NOT)을 이용해 결합할 수 있습니다.
{: .prompt-question id="함께-사용하기-bool"}

```json
  {
    "query": {
      "bool": {
        "must": [
          { "match": { "title": "Elasticsearch" } }  // 검색어 포함 (점수 계산 O)
        ],
        "filter": [
          { "term": { "category": "search" } },  // 특정 카테고리 필터링 (점수 계산 X, 캐싱 O)
          { "range": { "published_date": { "gte": "2024-01-01" } } } // 날짜 필터링 (점수 계산 X)
        ]
      }
    }
  }
```


동작 방식
: 위 내용과 같이, bool 쿼리로 filter와 must을 조합해 쿼리시 대략적으로 어떻게 검색하는지 알아보자.

1. 내부적으로 실행 우선순위를 최적화하여 검색 성능을 높인다.
: 필터링 가능한 부분은 먼저 적용해 성능을 최적화 하려고 합니다.

2. filter 조건을 평가하여, 문서 집합을 축소합니다
: 위의 예제에서는 [category=search와 published_date >= 2024-01-01]을 만족하는 문서를 필터링한다.
: 이 필터가 자주 사용되는 경우 캐싱이 발생한다.

3. must(match) 조건을 적용하여 `_score` 계산합니다.
: 필터링된 문서들만 대상으로 "title": "Elasticsearch" 일치 여부를 평가하며, `_score`를 계산한다.

