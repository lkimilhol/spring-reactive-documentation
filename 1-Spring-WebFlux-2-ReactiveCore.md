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


다음 표는 서버 종속성을 설명합니다 (지원되는 버전도 참조).

|서버 이름|그룹 ID|Artifact name|
|------|---|---|
|Reactor Nettyio.projectreactor.netty|reactor-netty|
|Undertow|io.undertow|undertow-core|
|Tomcat|org.apache.tomcat.embed|tomcat-embed-core|
|Jetty|org.eclipse.jetty|jetty-server, jetty-servlet|

아래 코드 스 니펫은 각 서버 API와 함께 HttpHandler 어댑터를 사용하여 표시됩니다.

Reactor Netty
```
HttpHandler handler = ...
ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(handler);
HttpServer.create().host(host).port(port).handle(adapter).bind().block();
```

Undertow
```
HttpHandler handler = ...
UndertowHttpHandlerAdapter adapter = new UndertowHttpHandlerAdapter(handler);
Undertow server = Undertow.builder().addHttpListener(port, host).setHandler(adapter).build();
server.start();
```

Tomcat
```
HttpHandler handler = ...
Servlet servlet = new TomcatHttpHandlerAdapter(handler);

Tomcat server = new Tomcat();
File base = new File(System.getProperty("java.io.tmpdir"));
Context rootContext = server.addContext("", base.getAbsolutePath());
Tomcat.addServlet(rootContext, "main", servlet);
rootContext.addServletMappingDecoded("/", "main");
server.setHost(host);
server.setPort(port);
server.start();
```

Jetty
```
HttpHandler handler = ...
Servlet servlet = new JettyHttpHandlerAdapter(handler);

Server server = new Server();
ServletContextHandler contextHandler = new ServletContextHandler(server, "");
contextHandler.addServlet(new ServletHolder(servlet), "/");
contextHandler.start();

ServerConnector connector = new ServerConnector(server);
connector.setHost(host);
connector.setPort(port);
server.addConnector(connector);
server.start();
```

### Servlet 3.1+ Container

서블릿 3.1+ 컨테이너에 WAR로 배포하려면 AbstractRestiveWebInitializer를 확장하여 WAR에 포함하면 됩니다. 이 클래스는 ServletHttpHandlerAdapter로 HttpHandler를 포장하고 Servlet으로 등록합니다.

## 1.2.2. WebHandler API
org.springframework.web.server 패키지는 HttpHandler 제공에 기반하여 여러 WebExceptionHandler, 여러 WebFilter 및 단일 WebHandler 구성 요소의 체인을 통해 요청을 처리하기 위한 범용 웹 API를 제공합니다. 구성 요소가 자동으로 감지되는 Spring Application Context를 가리키거나 구성 요소를 Builder에 등록함으로써 체인을 WebHttpHandlerBuilder와 함께 구성할 수 있습니다.

### 특별한 빈 타입

아래 표에는 WebHttpHandlerBuilder가 Spring ApplicationContext에서 자동 탐지하거나 직접 등록할 수 있는 구성 요소가 나열되어 있습니다.

|빈 이름|빈 타입|카운트|설명|
|------|---|---|---|
|<any\>|webExceptionHandler|0..N|WebFilter 인스턴스 체인과 대상 WebHandler의 예외 처리를 제공합니다. 자세한 내용은 예외를 참조하십시오.|
|<any\>|WebFilter|0..N|필터 체인의 나머지 부분과 대상 WebHandler 앞뒤에 가로채기 스타일 논리를 적용합니다. 자세한 내용은 필터를 참조하십시오.|
|webHandler|webHandler|1|요청을 위한 핸들러 입니다.||
|webSessionManager|webSessionManager|0..1|ServerWebExchange의 메서드를 통해 노출된 WebSession 인스턴스에 대한 관리자. 디폴트는 DefaultWebSessionManager 입니다.|
|serverCodecConfigurer|ServerCodecConfigurer|0..1|폼 데이터와 멀티 파트 데이터를 구문 분석하기 위한 HttpMessageReader 인스턴스에 액세스하려면 ServerWebExchange의 메소드를 통해 노출됩니다. ServerCodecConfigure.create()를 기본으로 합니다.|
|localeContextResolver|LocaleContextResolver|0..1|ServerWebExchange의 메서드를 통해 노출된 LocaleContext 확인 메서드입니다. 기본적으로 HeaderLocaleContextResolver를 수락합니다.|
|forwardedHeaderTransformer|ForwardedHeaderTransformer|0..1|전달된 형식 헤더를 처리하려면 헤더를 추출하여 제거하거나 헤더만 제거하십시오. 기본적으로 사용되지 않음.|

### 폼 데이터

ServerWebExchange는 form data에 억세스 하는 다음 방법을 제공합니다.

```
Mono<MultiValueMap<String, String>> getFormData();
```

DefaultServerWebExchange는 구성된 HttpMessageReader를 사용하여 양식 데이터(application/x-www-form-urlencoded)를 MultiValueMap으로 구문 분석합니다. 기본적으로 FormHttpMessageReader는 ServerCodecConfigurerin에서 사용하도록 구성됩니다(Web Handler API 참조).

### 멀티파트 데이터

ServerWebExchange는 멀티파트 데이터에 액세스하는 다음 방법을 제공합니다.

```
Mono<MultiValueMap<String, Part>> getMultipartData();
```

DefaultServerWebExchange는 구성된 HttpMessageReader<MultiValueMap<String, Part>를 사용하여 멀티파트/폼데이터 콘텐츠를 멀티밸류맵으로 구문 분석합니다. 기본적으로 이것은 타사 종속성이 없는 DefaultPartHttpMessageReader입니다. 또는 SynchronossPartHttpMessageReader를 사용할 수 있으며, 이는 Synchronoss NIO Multipart 라이브러리를 기반으로 합니다. 둘 다 ServerCodecConfigurerin을 통해 구성됩니다(Web Handler API 참조).

스트리밍 방식으로 다중 부분 데이터를 구문 분석하려면 HttpMessageReader<Part>에서 반환된 Flux<Part>를 대신 사용할 수 있습니다. 예를 들어 어노테이션 컨트롤러에서 @RequestPart를 사용하면 지도와 같은 기능을 사용할 수 있습니다.


### Forwarded Headers

요청이 로드 밸런서와 같은 프록시를 통과하면 호스트, 포트 및 스키마가 변경될 수 있습니다. 따라서 클라이언트의 관점에서 올바른 호스트, 포트 및 스키마를 가리키는 링크를 만드는 것이 어렵습니다.

RFC 7239는 프록시가 원래 요청에 대한 정보를 제공하는 데 사용할 수 있는 Forwarded HTTP 헤더를 정의합니다. X-Forwarded-Host, X-Forwarded-Port, X-Forwarded-Proto, X-Forwarded-Ssl 및 X-Forwarded-Prefix를 포함한 다른 비표준 헤더도 있습니다.

ForwardedHeaderTransformer는 전달된 헤더를 기준으로 요청의 호스트, 포트 및 스키마를 수정한 다음 해당 헤더를 제거하는 구성 요소입니다. Forwarded Header Transformer라는 이름으로 빈으로 선언하여 감지되어 사용됩니다.

전달된 헤더에 대한 보안 고려 사항이 있습니다. 프로그램이 헤더가 프록시, 의도한 대로 추가되었는지 또는 악의적인 클라이언트에 의해 추가되었는지 알 수 없기 때문입니다. 따라서 신뢰 경계에 있는 프록시가 외부에서 들어오는 신뢰할 수 없는 전달된 트래픽을 제거하도록 구성되어야 합니다. 또한 ForwardedHeaderTransformer를 removeOnly=true로 구성할 수 있습니다. 이 경우 헤더는 제거되지만 사용되지 않습니다.

> 5.1에서는 ForwardedHeaderFilter가 더 이상 사용되지 않으며 ForwardedHeaderTransformer로 대체되어 교환이 생성되기 전에 ForwardedHeader를 더 일찍 처리할 수 있습니다. 필터가 구성된 경우 필터 목록에서 제거되고 대신 ForwardedHeaderTransformer가 사용됩니다.


## 1.2.3. Filters

WebHandler API에서 WebFilter를 사용하여 필터의 나머지 처리 체인과 대상 WebHandler의 앞뒤에 인터셉트 스타일 로직을 적용할 수 있습니다. WebFlux Config를 사용할 때 WebFilter를 등록하는 것은 스프링빈으로 선언하고 (선택적으로) 빈 선언에 @Order를 사용하거나 Ordered를 구현하여 우선순위를 표현하는 것만큼 간단합니다.

자세한 내용은 CORS 및 CORS WebFilter에 대한 섹션을 참조하십시오.

## 1.2.4. Exceptions

