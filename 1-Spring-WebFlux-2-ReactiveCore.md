# 1.2.1 리액티브 코어

스프링웹 모듈은 다음과 같은 리액티브 웹 어플리케이션 기본 지원이 포함되어 있습니다.

- 서버 요청 처리에는 두 가지 수준의 지원이 있습니다.
  - HttpHandler: Reactor Netty, Undertow, Tomcat, Jetty 및 모든 Servlet 3.1 + 컨테이너 용 어댑터와 함께 논블로킹 I/O 및 Reactive Streams 백프래셔으로 HTTP 요청 처리를위한 기본 지원.
  - WebHandler API: 약간 더 높은 수준의 요청 처리를위한 범용 웹 API이며 어노테이션 컨트롤러 및 기능 엔드 포인트와 같은 구체적인 프로그래밍 모델이 구축되어 있습니다.

- 클라이언트 측에서는 Reactor Netty, Reactive Jetty HttpClient 및 Apache HttpComponents용 어댑터와 함께 논블로킹 I/O 및 Reactive Streams 백프래셔로 HTTP 요청을 수행하기 위한 기본 ClientHttpConnector 지원이 있습니다. 응용 프로그램에 사용되는 더 높은 수준의 WebClient는 기본으로 기반합니다.
- 클라이언트 및 서버의 경우 HTTP 요청 및 응답 내용의 직렬화 및 역직렬화를 위한 코덱입니다.

## 1.2.1 HttpHandler

HttpHandler는 단건 요청과 응답을 다루는 간단한 방법 입니다. 의도적으로 간소화 되어 있고, 주요 목적은 다른 HTTP 서버 API에 대한 최소한의 추상화 입니다.

다음 표에서는 지원되는 서버 API에 대해 설명합니다.

|서버 이름|서버 API 사용|리액티브 스트림즈 지원|
|------|---|---|
|Netty|Netty API|Reactor Netty|
|Undertow|Undertow API|스프링웹: Undertow에서 리액티브 스트림즈의 브릿지|
|Tomcat|서블릿 3.1 논블락킹 I/O; 톰캣 API 는 ByteBuffers vs byte[]를 읽고 씁니다|스프링웹: 서블릿 3.1 논블락킹 I/O에서 리액티브 스트림즈로의 브릿지|
|Jetty|서블릿 3.1 논블락킹 I/O; Jetty API 는 ByteBuffers vs byte[]를 읽고 씁니다|스프링웹: 서블릿 3.1 논블락킹 I/O에서 리액티브 스트림즈로의 브릿지|
|서블릿 3.1 컨테이너|서블릿 3.1 논블락킹 I/O|스프링웹: 서블릿 3.1 논블락킹 I/O에서 리액티브 스트림즈로의 브릿지|


