# 1.2 리액티브 코어

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

### CORS

Spring WebFlux는 컨트롤러의 어노테이션을 통해 CORS 구성을 세밀하게 지원합니다. 그러나 Spring Security와 함께 사용할 때는 내장 CorsFilter를 사용하는 것이 좋습니다.이 필터는 Spring Security의 필터 체인보다 먼저 주문해야합니다.

## 1.2.4. Exceptions

WebHandler API에서는 WebExceptionHandler를 사용하여 WebFilter 인스턴스 체인과 대상 WebHandler의 예외를 처리할 수 있습니다. WebFlux Config를 사용할 때 WebExceptionHandler를 등록하는 것은 Spring bean으로 선언하고 bean 선언에 @Order를 사용하거나 Ordered를 구현하여 우선 순위를 표현하는 것만 큼 간단합니다.

다음 표는 사용 가능한 WebExceptionHandler 구현을 설명합니다.

|예외 핸들러|설명|
|------|---|
|ResponseStatusExceptionHandler|예외의 HTTP 상태 코드로 응답을 설정하여 ResponseStatusException 유형의 예외를 처리합니다.|
|WebFluxResponseStatusExceptionHandler|예외에 대한 @ResponseStatus 주석의 HTTP 상태 코드를 결정할 수 있는 ResponseStatusExceptionHandler의 확장입니다.이 처리기는 WebFlux 구성에 선언됩니다.|

1.2.5. Codecs

스프링 웹 및 스프링 코어 모듈은 리액티브 스트림 백프레셔를 사용하여 논블로킹 I/O를 통해 더 높은 수준의 개체와 바이트 콘텐츠를 직렬화하고 역직렬화할 수 있도록 지원합니다. 다음은 이러한 지원에 대해 설명합니다.

- 인코더와 디코더는 HTTP와 무관하게 콘텐츠를 인코딩하고 디코딩하는 저수준 계약이다.

- HttpMessageReader 및 HttpMessageWriter는 HTTP 메시지 내용을 인코딩하고 디코딩하는 약속입니다.

- Encoder는 EncoderHttpMessageWriter로 래핑하여 웹 응용 프로그램에서 사용하도록 조정할 수 있으며 Decoder는 DecoderHttpMessageReader로 래핑할 수 있습니다.

- 데이터 버퍼는 다른 바이트 버퍼 표현(예를 들어 Netty ByteBuf, java.nio.ByteBuffer 등)을 추상화하며 모든 코덱이 작업하는 것입니다. 자세한 내용은 "스프링 코어" 섹션의 데이터 버퍼 및 코덱을 참조하십시오.

스프링 코어 모듈은 byte[], ByteBuffer, DataBuffer, Resource 및 String 인코더 및 디코더 구현을 제공합니다. 스프링 웹 모듈은 폼 데이터, 멀티파트 콘텐츠, 서버 전송 이벤트 등을 위한 웹 전용 HTTP 메시지 리더 및 작성기 구현과 함께 잭슨 JSON, 잭슨 스마일, JAXB2, 프로토콜 버퍼 및 기타 인코더와 디코더를 제공합니다.

ClientCodecConfigurer 및 ServerCodecConfigurer는 일반적으로 응용 프로그램에서 사용할 코덱을 구성하고 사용자 지정하는 데 사용됩니다. HTTP 메시지 코덱 구성에 대한 절을 참조하십시오.

### Jackson JSON

JSON과 이진 JSON(Smile)은 모두 Jackson 라이브러리가 있을 때 지원됩니다.

Jackson2Decoder는 다음과 같이 작동합니다.

- Jackson의 비동기 논블록킹 파서는 바이트 청크 스트림을 JSON 객체를 나타내는 TokenBuffer의 스트림으로 집계하는 데 사용됩니다.

- 각 TokenBuffer는 Jackson의 ObjectMapper로 전달되어 더 높은 수준의 객체를 만듭니다.

- 단일 값 게시자(예: 모노)로 디코딩할 때 토큰 버퍼가 하나 있습니다.

- 멀티 밸류 게시자로 디코딩 할 때 (예. Flux), 각 TokenBuffer는 완전히 형성된 객체에 대해 충분한 바이트가 수신되는 즉시 ObjectMapper로 전달됩니다. 입력 콘텐츠는 JSON 배열 또는 NDJSON, JSON 라인 또는 JSON 텍스트 시퀀스와 같은 라인으로 구분된 JSON 형식일 수 있습니다.

Jackson2Encoder는 다음과 같이 작동합니다:

- 단일 값 게시자(예: Mono)의 경우 ObjectMapper를 통해 직렬화하기만 하면 됩니다.

- application/json이 있는 멀티 벨류 퍼블리셔의 경우 기본적으로 Flux#collectToList()를 사용하여 값을 수집한 다음 결과 컬렉션을 직렬화합니다.

- application/x-ndjson 또는  application/stream + x-jackson-smile과 같은 스트리밍 미디어 유형이 있는 멀리 밸류 퍼블리셔의 경우 줄로 구분된 JSON 형식을 사용하여 각 값을 개별적으로 인코딩, 쓰기 및 플러시합니다. 다른 스트리밍 미디어 타입들이 인코더에 등록될 수 있습니다.

- SSE의 경우 이벤트별로 Jackson2Encoder가 호출되고 지연 없이 전달되도록 출력이 플러시됩니다.

> 기본적으로 Jackson2Encoder와 Jackson2Decoder는 String 유형의 요소를 지원하지 않습니다. 대신 문자열 또는 문자열 시퀀스가 CharSequenceEncoder에 의해 렌더링되는 직렬화된 JSON 콘텐츠를 나타낸다는 것이 기본 가정이다. Flux<String>에서 JSON 배열을 렌더링해야 하는 경우 Flux#collectToList()를 사용하여 Mono<List<String>를 인코딩합니다.

### Form Data

FormHttpMessageReader 및 FormHttpMessageWriter는 응용 프로그램/x-www-form-url 인코딩된 콘텐츠를 디코딩하고 인코딩할 수 있도록 지원합니다.서버 웹 익스체인지(ServerWebExchange)는 여러 위치에서 양식 내용에 액세스해야 하는 서버 측에서 FormHttpMessageReader를 통해 내용을 구문 분석한 다음 반복 액세스를 위해 결과를 캐시하는 전용 getFormData() 메서드를 제공합니다. WebHandler API 섹션의 Form Data를 참조하십시오.getFormData()를 사용하면 원본 원시 콘텐츠를 더 이상 요청 본문에서 읽을 수 없습니다. 이러한 이유로, 응용프로그램은 원시 요청 본문에서 읽기보다는 캐시된 양식 데이터에 액세스하기 위해 서버 웹 Exchange를 일관되게 통과해야 합니다.

### Multipart

MultipartHttpMessageReader 및 MultipartHttpMessageWriter는 "Multipart/form-data" 콘텐츠의 디코딩 및 인코딩을 지원합니다. 차례로 MultipartHttpMessageReader는 다른 HttpMessageReader에게 Flux<Part>에 대한 실제 구문 분석을 위임한 다음 단순히 부품을 MultiValueMap으로 수집합니다. 기본적으로 DefaultPartHttpMessageReader가 사용되지만 ServerCodecConfigurer를 통해 변경할 수 있습니다. DefaultPartHttpMessageReader에 대한 자세한 내용은 DefaultPartHttpMessageReader의 javadoc을 참조하십시오.

서버 웹 익스체인지에서는 멀티파트 양식 컨텐츠에 여러 위치에서 액세스해야 하는 서버 측에서 MultipartHttpMessageReader를 통해 컨텐츠를 구문 분석한 다음 반복 액세스를 위해 결과를 캐시하는 전용 getMultipartData() 메서드를 제공합니다. WebHandler API 섹션의 멀티파트 데이터를 참조하십시오.

getMultipartData()를 사용하면 원본 원시 콘텐츠를 더 이상 요청 본문에서 읽을 수 없습니다. 이러한 이유로, 어플리케이션은 부품에 대한 반복적이고 지도와 같은 액세스에 대해 getMultipartData()를 일관되게 사용하거나, 그렇지 않으면 SynchronossPartHttpMessageReader를 사용하여 Flux<Part>에 한 번 액세스해야 합니다.

### Limits

입력 스트림의 일부 또는 전부를 버퍼링하는 디코더 및 HttpMessageReader 구현체는 메모리에서 버퍼링할 최대 바이트 수를 제한하여 구성할 수 있습니다. 예를 들어 @RequestBody byte[], x-ww-form-url 인코딩된 데이터가 있는 컨트롤러 메서드와 같이 입력이 집계되고 단일 개체로 표현되기 때문에 버퍼링이 발생하는 경우가 있습니다. 버퍼링은 입력 스트림을 분할할 때(예: 구분된 텍스트, JSON 객체의 스트림 등) 스트리밍에서도 발생할 수 있습니다. 이러한 스트리밍 사례의 경우 스트림의 한 개체와 관련된 바이트 수에 제한이 적용됩니다.

버퍼 크기를 설정하려면 지정된 디코더 또는 HttpMessageReader가 maxInMemorySize 속성을 노출하는지 확인하고, 노출되어 있는 경우 Javadoc에 기본값에 대한 세부 정보가 표시됩니다. 서버 측에서 ServerCodecConfigurer는 모든 코덱을 설정할 수 있는 단일 위치를 제공합니다. HTTP 메시지 코덱을 참조하십시오. 클라이언트 측에서는 WebClient에서 모든 코덱에 대한 제한을 WebClient.Builder에서 변경할 수 있습니다.

Multipart 구문 분석의 경우 maxInMemorySize 속성은 파일이 아닌 부분의 크기를 제한합니다. 파일 요소의 경우 요소가 디스크에 기록되는 임계값을 결정합니다. 디스크에 기록된 파일 부분의 경우 파트 당 디스크 공간의 양을 제한하는 추가 maxDiskUsagePerPart 속성이 있습니다. 또한 multipart 요청의 전체 부품 수를 제한하는 maxParts 속성이 있습니다. WebFlux에서 세 가지를 모두 구성하려면 미리 구성된 MultipartHttpMessageReader 인스턴스를 ServerCodecConfigurer에 제공해야 합니다.

### Streaming

HTTP 응답(예: text/event-stream, application/x-ndjson)으로 스트리밍할 때는 연결이 끊긴 클라이언트를 안정적으로 감지하기 위해 데이터를 정기적으로 전송하는 것이 중요합니다. 이러한 전송은 댓글 전용 빈 SSE 이벤트 또는 하트비트 역할을 효과적으로 수행할 수 있는 다른 "no-op" 데이터일 수 있습니다.

### DataBuffer

DataBuffer는 WebFlux의 바이트 버퍼를 나타냅니다. 이 참조의 스프링 코어 부분에 대한 자세한 내용은 데이터 버퍼 및 코덱에 대한 섹션을 참조하십시오. Netty와 같은 일부 서버에서는 바이트 버퍼가 풀링되고 참조 카운트가 이루어지며 메모리 누수를 방지하기 위해 소비될 때 해제되어야 합니다.

웹플럭스 어플리케이션은 코덱에 의존하여 더 높은 수준의 개체로 변환하거나 사용자 지정 코덱을 만들지 않는 한, 데이터 버퍼를 직접 소비하거나 생산하지 않는 한, 일반적으로 이러한 문제에 대해 걱정할 필요가 없습니다. 이러한 경우 데이터 버퍼 및 코덱, 특히 데이터 버퍼 사용에 대한 섹션을 참조하십시오.

## 1.2.6. Logging

Spring WebFlux의 DEBUG 레벨 로깅은 소형, 최소 및 인간 친화적으로 설계되었습니다. 특정 문제를 디버깅할 때만 유용한 다른 정보와 반복해서 유용한 정보의 높은 가치 비트에 중점을 둡니다.

TRACE 수준 로깅은 일반적으로 DEBUG와 동일한 원칙을 따르지만(예를 들어 방화벽이 되어서는 안 된다) 문제를 디버깅하는 데 사용할 수 있습니다. 또한 일부 로그 메시지는 TRACE와 DEBUG에서 서로 다른 수준의 세부 정보를 표시할 수 있습니다.

좋은 로깅은 로그를 사용하는 경험에서 비롯됩니다. 만약 당신이 명시된 목표를 달성하지 못하는 것을 발견한다면, 우리에게 알려주세요.

### Log Id

WebFlux에서 단일 요청은 여러 스레드에서 실행될 수 있으며 스레드 ID는 특정 요청에 속하는 로그 메시지를 상호 연결하는 데 유용하지 않습니다. 따라서 WebFlux 로그 메시지에는 기본적으로 요청별 ID가 접두사로 지정됩니다.

서버 측에서 로그 ID는 ServerWebExchange 속성 (LOG_ID_ATTRIBUTE)에 저장되며 해당 ID를 기반으로하는 완전 형식의 접두어는 ServerWebExchange # getLogPrefix ()에서 사용할 수 있습니다. WebClient 측에서는 log ID가 ClientRequest 속성(LOG_ID_ATTRIBUTE)에 저장되지만 ClientRequest#logPrefix()에서는 완전히 포맷된 접두사를 사용할 수 있습니다.

### Sensitive Data

DEBUG 및 TRACE 로깅은 중요한 정보를 기록할 수 있습니다. 따라서 양식 매개 변수와 헤더는 기본적으로 마스킹되며 해당 로깅을 완전히 사용하도록 명시적으로 설정해야 합니다.

다음 예제에서는 서버 측 요청에 대해 그렇게 하는 방법을 보여 줍니다.

```java
@Configuration
@EnableWebFlux
class MyConfig implements WebFluxConfigurer {

    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        configurer.defaultCodecs().enableLoggingRequestDetails(true);
    }
}
```

다음 예제에서는 클라이언트 측 요청에 대해 이렇게 하는 방법을 보여 줍니다.

```java
Consumer<ClientCodecConfigurer> consumer = configurer ->
        configurer.defaultCodecs().enableLoggingRequestDetails(true);

WebClient webClient = WebClient.builder()
        .exchangeStrategies(strategies -> strategies.codecs(consumer))
        .build();
```

### Appenders

SLF4J 및 Log4J2와 같은 로깅 라이브러리는 차단을 방지하는 비동기 로거를 제공합니다. 로깅을 위해 대기열에 넣을 수 없는 메시지를 삭제하는 등의 단점이 있지만, 현재 reactive, 논블로킹 애플리케이션에서 사용할 수 있는 최고의 옵션입니다.

### Custom codecs

어플리케이션은 추가 미디어 타입을 지원하기 위해 사용자 정의 코덱을 등록하거나 기본 코덱에서 지원하지 않는 특정 동작을 등록할 수 있습니다. 개발자가 표현한 일부 구성 옵션은 기본 코덱에 적용됩니다. 사용자 지정 코덱은 버퍼 제한을 적용하거나 중요한 데이터를 기록하는 것과 같은 기본 설정에 맞출 기회를 얻기를 원할 수 있습니다.다음 예제에서는 클라이언트 측 요청에 대해 이렇게 하는 방법을 보여 줍니다.

```java
WebClient webClient = WebClient.builder()
        .codecs(configurer -> {
                CustomDecoder decoder = new CustomDecoder();
                configurer.customCodecs().registerWithDefaultConfig(decoder);
        })
        .build();
```