---
title: Java Standard - 네트워킹(Networking)
description: >-
    ...
author: ggong
date: 2023-02-24 17:36:32 +0900
categories: [Book,Java Standard]
tags: [book,java-standard,study]
pin: false
media_subpath: '/assets/img/_posts/book/java-standard'
references:
  - name: ""
    url: ""
---


## 네트워킹

- 의미: 두 대 이상의 컴퓨터를 케이블로 연결하여 네트워크를 구성하는 것
- 발전
  - 초기: 단 몇대의 컴퓨터로 구성
  - 현재: 셀 수 없이 많은 컴퓨터들로 네트워크를 구성

### 클라이언트/서버

- 클라이언트/서버: 컴퓨터간의 관계를 역할로 구분하는 개념
  - 클라이언트: 서비스를 제공하는 컴퓨터
  - 서버: 서비스를 사용하는 컴퓨터
  - 서비스: 서버가 클라이언트로부터 요청받은 작업을 처리하여 그 결과를 제공하는 것
- 서버기반 모델: 전용서버를 두는 것
  - 안정적인 서비스 제공 가능
  - 공유 데이터의 관리와 보안이 용이
  - 서버구축비용과 관리비용이 듬
- P2P 모델: 별도의 전용 서버 없이 각 클라이언트가 서버 역할을 동시에 수행하는 것
  - 서버구축 및 운용비용 절감
  - 자원의 활용을 극대화
  - 자원의 관리가 어려움
  - 보안이 취약

### IP 주소

- 의미: 컴퓨터(host)를 구별하는데 사용되는 고유한 값
  - 구성: 4byte(32bit)의 정수로 `a.b.c.d` 형식
  - 네트워크 구성에 따라 네트워크 주소와 호스트 주소의 비율이 달라짐
  - IP 주소의 네트워크 주소가 같다는 것=호스트가 같은 네트워크에 포함되어 있는 것
- IP주소와 서브넷 마스크의 관계
  
    ![Build source](16-1.png){:.normal w="500"}

  - IP 주소와 서브넷 마스크를 비트연산자 `&` 로 연산 = IP 주소에서 네트워크 주소만 추출됨
  - 이를 통해서 같은 네트워크 상에 존재하는지 확인 할 수 있음

> 호스트 주소
> 0: 네트워크 자기자신을 의미
> 255: 브로드캐스트 주소로 사용
> 실제로 네트워크에 포함 가능한 호스트의 개수는 254개
{:.prompt-question}

### InetAddress

- 의미: 자바에서 제공하는 IP주소를 다루기 위한 클래스
- 메서드
  
  ![Build source](16-2.png){:.normal w="500"}


### URL

- 의미: 인터넷에 존재하는 여러 서버들이 제공하는 자원에 접근할 수 있는 주소를 표현
- 형태

    ```jsx
    http://www.codechobo.com:80/sample/hello.html?referer=codechobo#index1
    프로토콜://호스트명:포트번호/경로명/파일명?쿼리스트링#참조
    ```

  - 프로토콜: 자원에 접근하기 위해 서버와 통신하는데 사용되는 통신규약 `http`
  - 호스트명: 자원을 제공하는 서버의 이름 `www.codechobo.com`
  - 포트번호: 통신에 사용되는 서버의 포트번호 `80`
  - 경로명: 접근하려는 자원이 저장된 서버상의 위치 `/sample/`
  - 파일명: 접근하려는 자원의 이름 `hello.html`
  - 쿼리: URL에서 ? 이후의 부분 `referer=codechobo`
  - 참조: URL에서 ## 이후의 부분 `index1`
- URL 클래스: 자바에서 제공하는 URL을 다루기 위한 클래스
  - 메서드
    
    ![Build source](16-3.png){:.normal w="500"}


### URLConnection

- 의미
  - 어플리케이션과 URL간의 연결 통신을 나타내는 최상위 클래스, 추상 클래스
  - 구현 클래스는 `HttpURLConnection`과 `JarURLConnection`이 있음
  - 연결하고자 하는 자원에 접근하고, 읽고 쓰기를 할 수 있음
- 메서드
   
  ![Build source](16-4.png){:.normal w="500"}
  ![Build source](16-5.png){:.normal w="500"}

- 특징
  - 연결 한 뒤에는 stream을 얻고, 이 이후부터는 파일에서 데이터를 읽는 것과 같음
  - URL이 유효하지 않으면 `Malformed - URLException` 발생

## 소켓 프로그래밍

- 의미: 소켓을 이용한 통신 프로그래밍을 의미
- Socket(소켓)
  - 프로세스간의 통신에 사용되는 양쪽 끝단을 의미
  - 소켓통신에 사용되는 프로토콜에 따라 다른 종류의 소켓을 구현하여 제공함

### TCP와 UDP

- 의미: `TCP`와 `UDP`모두 `TPC/IP` 프로토콜에 포함되어 있음, OSI 7 계층 중 전송계층에 해당
  - TCP
    - 연결기반: 연결 후 통신, 1:1 통신
    - 데이터의 경계를 구분함
    - 신뢰성 있는 데이터 전송
      - 전송 순서 보장, 수신 여부 확인(손실되면 재전송), 패킷을 관리할 필요가 없음
    - UDP 보다 전송속도가 느림
  - UDP
    - 비연결 기반: 연결없이 통신, 1:1 1:n n:n 통신
    - 데이터의 경계를 구분 안함
    - 신뢰성 없는 데이터 전송
      - 전송 순서 바뀔 수 있음, 수신여부 확인 안함(손실되어도 확인 못함), 패킷관리를 해야함
    - TCP보다 전송속도가 빠름

### TCP소켓 프로그래밍

- 의미: 클라이언트와 서버의 1:1 통신
- 특징
  - 서버 프로그램이 클라이언트보다 먼저 실행되어 클라이언트를 기다리고 있어야함
  - `Socket`과 `ServerSocket`을 사용해 통신함
- 통신과정
  1. `서버 프로그램`: 서버소켓 사용해 서버 컴퓨터의 특정 포트에서 클라이언트의 연결요청을 대기

     ![Build source](16-6.png){:.normal w="500"}
     _서버 프로그램 실행, 서버 소켓 생성_
      
  2. `클라이언트 프로그램`: 접속할 서버의 IP와 포트정보를 가지고 소켓을 생성 → 연결을 요청

     ![Build source](16-7.png){:.normal w="500"}
     _클라이언트 프로그램의 소켓 생성 →  서버 프로그램에 연결 요청_
      
  3. `서버 소켓`: 클라이언트의 요청을 받으면 서버에 새로운 소켓을 생성, `클라이언트 소켓`과 연결

     ![Build source](16-8.png){:.normal w="500"}
     _서버에 새로운 소켓 생성, 클라이언트 프로그램의 소켓과 새로 만든 소켓과 연결_
     ![Build source](16-9.png){:.normal w="500"}

     - 서버 소켓과 소켓의 차이점
         - `서버 소켓`의 역할: 포트와 결합되어, 요청이 들어올 때 마다 새로운 소켓을 생성하고, 요청한 소켓과 연결하는 것
         - `소켓`의 역할: 프로세스간의 통신을 담당(실제 데이터를 주고 받음), 입출력스트림을 가지고 있어 이를 통해 통신이 이루어짐
         - `소켓`은 한 포트를 공유해서 통신이 가능하지만, `서버 소켓`은 포트를 독점함
         : 서버 소켓이 포트를 공유한다면, 클라이언트 소켓에게 요청이 왔을 때 어떤 서버 프로그램에게 요청을 보낸 것인지 알 수 없기 때문에 독점함
          프로토콜을 다르게 사용하는 경우라면 가능함, 프로토콜로 구분이 가능하기 때문
              
  4. `서버 소켓` - `클라이언트 소켓` 의 1:1 통신
  
     ![Build source](16-10.png){:.normal w="500"}
     _실제 데이터 교환은 입출력 스트림(한쪽 출력 스트림은 다른 한쪽의 입력스트림으로 연결)_

- 예제
  - 서버

      ```java
      ServerSocket serverSocket = null;
      try{
        // 7777번 포트번호로 서버 소켓 생성
        serverSocket = new ServerSocket(7777);
      } catch(IOException e){ e.printStackTrace(); }
      
      while(true){
        try{
          // 클라이언트 소켓의 요청이 들어오면
          // 클라이언트 소켓과 통신할 새로운 소켓을 생성
          Socket socket = serverSocket.accept();
      
          // 소켓의 출력스트림을 얻음 -> 이걸 통해서 데이터를 보냄
          OutputStream out = socket.getOutputStream();
          DataOutputStream dos = new DataOutputStream(out);
      
          dos.writeUTF("서버 소켓-> 클라이언트 소켓으로 보내는 데이터");
      
          // 스트림과 소켓을 닫아줌
          dos.close();
          socket.close();
        } catch(IOException e){ e.printStackTrace(); }
      }
      
      ```

  - 클라이언트 프로그램

      ```java
      try{
      
        // 소켓을 생성하여 서버 소켓에게 연결을 요청
        // IP와 port 번호를 가지고 생성하면 자동적으로 연결 요청
        Socket socket = new Socket("127.0.0.1", 7777);
        
        
        // 소켓의 입력스트림을 얻음 -> 이걸 통해서 데이터를 받음
        InputStream in = socket.getInputStream();
        DataInputStream dis = new DataInputStream(in);
      
        // 스트림과 소켓을 닫아줌
        dis.close();
        socket.close();
        } catch(IOException e){ e.printStackTrace(); }
      ```

- 메서드
  - `setSoTimeout(int timeout)`: 서버소켓의 대기시간을 지정
    - 지정 시간을 넘으면 accept()에서 `SocketTimeoutException`이 발생
  - `getPort()`, `getLocalPort()`: TPC/IP 통신에서 사용하고 있는 포트를 알아 낼 수 있음

### UDP소켓 프로그래밍

- 의미: 비연결지향 통신
- 특징
  - `DatagramSocket`과 `DatagramPacket`을 사용해 통신
  - `DatagramPacket` 에 데이터를 담아 보내면 `DatagramSocket` 에 도착함
- `DatagramPacket` 구조
  - 헤더 :`DatagramPacket`를 수신할 호스트의 정보(호스트의 주소와 포트)가 저장
  - 데이터: 전송할 데이터
- 예제
  - 클라이언트

      ```java
      //서버에 전송하기 위해 필요함
      DatagramSocket datagramSocket = new DatagramSocket();
      InetAddress serverAddress = InetAddress.getByName("127.0.0.1");
      
      // 데이터를 주고 받을 공간
      byte[] message = new byte[100];
      
      // datagramPacket생성
      DatagramPacket outPacket = new DatagramPacket(message, 1, serverAddress, 7777);
      DatagramPacket outPacket = new DatagramPacket(message, message.length);
      
      datagramSocket.send(outPacket);//데이터 전송
      datagramSocket.receive(inPacket);//데이터 수신
      
      datagramSocket.close();
      ```

  - 서버

      ```java
      DatagramSocket socket = new DatagramSocket(7777);
      DatagramPacket inPacket, outPacket;
      
      byte[] inMessage = new byte[10];
      byte[] outMessage;
      
      while(true){
        // 데이터 수신을 위해 packet을 생성
        inPacket = new DatagramPacket(inMessage, inMessage.length);
        
        // 패킷을 통해 데이터를 수신함
        socket.receive(inPacket);
      
        // 수신한 패킷을 통해 client의 IP주소와 Port를 얻음
        InetAddress clientAddress = inPacket.getAddress();
        int clientPort = inPacket.getPort();
      
        // 패킷을 생성해서 client에게 전송
        outPacket = new DatagramPacket(outMessage, outMessage.length, clientAddress, clientPort);
      ```

- 메서드
  - `getAddress()`: 클라이언트의 주소를 조회
  - `getPort()`: 클라이언트의 포트 번호를 조회
