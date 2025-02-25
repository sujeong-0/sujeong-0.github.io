---
title: Interceptor와 Filter
description: >-
  Interceptor와 Filter의 차이점을 알아보고, 각각 언제 사용하면 좋을지 알아보자.
author: ggong
date: 2024-11-03 16:02:00 +0900
categories: [ Study, Springboot Java]
tags: [ study, springboot, topic ]
pin: false
media_subpath: '/assets/img/_posts/study/springboot-java'
references:
  - name: "[Spring] 필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도 - (1)"
  - URL: "https://mangkyu.tistory.com/173"
---


## Interceptor

의미
: Spring이 제공하는 기술
: `디스패처 서블릿(Dispatcher Servlet)`이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공

Spring의 Application Context에 등록

**사용 예시**
: AOP를 흉내낼때
: Spring 애플리케이션에서 전역적으로 전/후처리 로직에서 예외를 사용하도록 할 때
: Handler Method에서 사용자의 권한을 체크해서 다른 동작을 시켜준다거나 할 때


> AOP와의 비교
> interceptor 대신 AOP를 적용하는 것도 가능하지만, 아래의 이유들로 컨트롤러의 호출 과정에 적용되는 부가기능들은 인터셉터를 사용하는 편이 낫다.
> - 컨트롤러는 타입과 실행 메소드가 모두 제각각이라 포인트컷(적용할 메소드 선별)의 작성이 어렵다.
> - 컨트롤러는 파라미터나 리턴 값이 일정하지 않다.
> - AOP에서는 HttpServletRequest/Response를 객체를 얻기 어렵지만 인터셉터에서는 파라미터로 넘어온다.
{: .prompt-info }


## Filter

의미
: J2EE 표준 스펙 기능
: `디스패처 서블릿(Dispatcher Servlet)`에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공
: `디스패처 서블릿(Dispatcher Servlet)`은 스프링의 가장 앞에 존재하는 프론트 컨트롤러이므로, 필터는 스프링 범위 밖에서 처리가 되는 것
    = 웹 컨테이너(ex. tomcat)에 의해 관리가 되는 것, 스프링 빈으로 등록은 됨

Web Application(Tomcat을 사용할 경우 web.xml)에 등록

사용 예시
: 보안 및 인증/인가 관련 작업
: 모든 요청에 대한 로깅 또는 검사
: 이미지/데이터 압축 및 문자열 인코딩
: Spring과 분리되어야 하는 기능


## Interceptor와 Filter 차이


![Build source](web-flow.png){: .light .normal}


1. 1~2 과정: `Handler Mapping`을 통해 요청에 대해 적절한 `Controller`를 찾는데 그 결과로 실행 체인(`HandlerExecutionChain`)을 얻는다.
실행 체인은 1개 이상의 `Interceptor`가 등록되어 있을 경우 순차적으로 `Interceptor`들을 거쳐 `controller`가 실행되도록 하고, 
`Interceptor`가 없을 경우 바로 `Controller`를 실행

2. 3     과정: `Dispatcher Servlet`은 실행할 `Controller` 정보를 `Handler Adapter`에게 전달, `Handler Adapter`는 전달받은 `Controller`를 실행하는데, 실행하기 이전에 `Interceptor`들을 먼저 실행
`Handler Adapter`가 `Controller` 메소드 혹은 `interceptor`를 실행하는 도중 `HttpSession`을 이용해야 한다면, 
`Servlet Container`가 `Session Storage`를 확인하여 `session`을 새로 발급하거나 기존의 `session`을 매핑

3. 4~5 과정: `controller`를 실행하기 전 처리해야 할 전처리 작업이나 요청 정보를 가공하거나 추가하는 작업을 진행
4. 14~15 과정: `controller`를 실행한 후 처리해야 할 후처리 작업


![Build source](interceptor-filter.png){: .light .normal}


## 요약
인터셉터와 필터는 **컨트롤러 실행 전에 요청을 가로챈다는 공통점**이 있지만, **동작 위치와 관리 주체가 다릅니다.**

필터(Filter)
: **웹 컨테이너(Servlet)가 관리**하며, **DispatcherServlet 실행 이전**에 동작합니다.
: **Spring과 무관한 요청도 처리 가능**하여 **인코딩 설정, 보안 검사, CORS 처리** 등에 사용됩니다.

인터셉터(Interceptor)
: **Spring이 관리**하며, **DispatcherServlet 이후, 컨트롤러 실행 직전에 동작**합니다.
: 주로 **사용자 인증, 권한 검증, API 요청 로깅** 등에 활용됩니다.

**웹 애플리케이션 전반에 영향을 주는 작업은 필터에서 처리하고, Spring 요청에만 적용할 로직은 인터셉터에서 처리**하는 것이 적절합니다.
