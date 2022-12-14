버전 5.3.23

이 문서에서는 Netty, Underow 및 Servlet 3.1+ 컨테이너와 같은 비블로킹 서버에서 실행할 수 있도록 Reactive Streams API를 기반으로 구축된 Reactive-Stack 웹 애플리케이션 지원에 대해 설명합니다. 개별 장에서는 Spring WebFlux 프레임워크, 대응형 WebClient, 테스트 지원 및 대응형 라이브러리를 다룹니다. 서블릿 스택 웹 응용 프로그램의 경우 서블릿 스택의 웹을 참조하십시오.

# 1. Spring WebFlux

스프링 프레임워크, 스프링 웹 MVC에 포함된 원래의 웹 프레임워크는 서블릿 API 및 서블릿 컨테이너 용으로 제작되었습니다. 리액티브 스택 웹 프레임워크인 스프링 웹플럭스는 버전 5.0 이후에서 추가 되었습니다. 리액티스 스트림즈 백 프레셔를 지원하며 네티, 언더토우 및 서블릿 3.1+ 컨테이너와 같은 서버에서 실행 됩니다.

이 두가지 웹 프레임워크는 소스 모듈의 이름을 미러링하고 스프링 프레임워크에 나란히 존재합니다. 각 모듈은 선택 사항입니다. 응용 프로그램은 하나 또는 다른 모듈을 사용하거나 경우에 따라 둘 다 사용할 수 있습니다. 예를 들어, 리액티브 웹클라이언트가 있는 스프링 MVC 컨트롤러 입니다.
-> 스프링 MVC 컨트롤러 내에서도 리액티브 웹 클라이어트가 사용 가능하다.

## 1.1 개요

왜 스프링 웹플럭스가 만들어졌을까요?

정답은 적은 수의 스레드로 동시서을 처리하고, 더 적은 하드웨어 리소스로 확장이 가능한 논블락킹 웹스택의 필요성 입니다. 서블릿 3.1은 논블록킹 I/O API를 지원했었습니다. 그러나, 이것을 사용하는 것은 동기 또는 블락킹 서블릿 API의 rest와 거리가 멉니다. 이것은 새로운 공통 API를 제공하는 동기가 되었습니다. 이것은 비동기, 논블락킹에 잘 설립된 서버 때문에 중요합니다. (예를 들어 Netty)

또 다른 답은 함수형 프로그래밍입니다. 자바5의 어노테이션이 기회를 만든것처럼(어노테이션이 있는 REST 컨트롤로나, 단위 테스트 등) 자바8의 람다 표현식은 자바에서 함수형 API를 위한 기회를 만들었습니다. 이는 논블록킹 애플리케이션, 연속적인 스타일 API(CompletableFuture 그리고 ReactiveX에 의해 알려짐)에 요긴합니다. 프로그래밍 모델 수준에서 자바8은 스프링 웹플럭스가 주석이 어노테이션이 달린 컨트롤러와 함께 기능적인 웹 엔드포인트를 제공 할 수 있게 하였습니다.

### 1.1.1 "Reactive" 정의

우리는 논블록킹, 함수형에 언급했습니다. 근데 리액티브의 의미는 무엇일까요?

리액티브의 용어는, 리액팅으로 변화되는 프로그래밍 모델을 (I/O 리액팅 이벤트 네트워크 요소, 마우스 이벤트 리액팅 UI 컨트롤러 등) 가르킵니다. 그런 의미로 논블락킹은 블락된 것 대신에 작업이 완료되거나 데이터를 사용할 수 있게 되기 때문에 리액팅입니다.

여기엔 또다른 중요한 메커니즘이 있는데, 스프링 팀은 리액티브 논블락킹 백프레셔와 연관이 있습니다. 동기식 명령형 코드에서 블락킹 콜은 호출자가 대기하도록 자연스러운 형태로 백프레셔 역할을 합니다. 논블록킹 코드에서는 퍼블리셔가 목적지를 압도하지 않도록 이벤트 발생률을 조절하는 것이 중요해집니다.

> 리액티브 스트림즈는 백프레셔를 동반한 비동기 사이의 컴포넌트를 정의한 작은 스펙입니다. 예를들어 데이터 레포지토리(퍼블리셔 역할)는 HTTP 서버(섭스크라이버 역할)가 응답에 쓸 수 있는 데이터를 생성 할 수 있습니다. 리액티브 스트림즈의 주된 목저은 얼마나 빠르고, 느리게 퍼블리셔가 데이터를 프로듀스 하는지 입니다.

### 1.1.2 Reactive API

리액티브 스트림즈는 상호 운용의 중요한 역할을 수행합니다. 이것은 라이브러리와 인프라스터럭쳐 컴포넌트들에 관심이 있지만, 어플리케이션 API에는 덜 유용적인데 왜냐하면 너무 low-level 이기 때문입니다. 어플리케이션들은 비동기 로직을 구성하기 위해 더욱 higher-level 그리고 풍부한 함수형 API를 필요로 합니다. - Java 8 Stream API와 유사하지만 컬렉션에만 해당되지 않습니다. 이것은 리액티브 라이브러리 역할입니다.

리액터는 스프링 웹플럭스가 선택한 리액티브 라이브러리 입니다. 이것은 0..1 그리고 0..N의 데이터 시퀀스에서 작업할 수 있는 모노와 플럭스 API 타입을 제공합니다. N(Flux)은 오퍼레이터의 ReactiveX 어휘와 정렬된 풍부한 오퍼레이터 집합을 통해 이루어집니다. 리액터는 리액티브 스트림즈 라이브러리이며 그러므로 모든 오퍼레이터가 논블록킹 백프래서를 지원합니다. 리액터는 서버사이드의 자바에 강력한 포커스를 가지고 있습니다. 이것은 스프링과의 콜라보래이션에 가깝도록 개발되어 졌습니다.

웹플럭스는 리액터에 코어 디펜던시를 필요로 하지만 이것은 리액티브 스트림즈를 통해 다른 리액티브 라이브러리와 상호 운용할 수 있습니다. 일반적인 룰에 따르면, 웹플럭스 API는 퍼블리셔를 인풋으로 수용하였으며 내부적으로는 리액터 유형에 적응하고 이를 사용하며 플럭스 또는 모노를 출력으로 반환합니다. 그래서 모든 퍼블리셔를 입력으로 전달하고 출력에 오퍼레이션을 적용 할 수 있지만 다른 리액티브 라이브러리와 함께 사용할 수 있도록 출력을 조정해야 합니다. 가능한 경우(예: 주석이 달린 컨트롤러), 웹플럭스는 RxJava 또는 다른 리액티브 라이브러리의 사용에 잘 적응합니다. 자세한 내용은 리액티브 라이브러리를 참조하세요.

### 1.1.3 Programming Models

스프링-웹 모듈은 HTTP 추상화, 서버를 지원하는 리액티브 스트림즈, 코덱들, 그리고 서블릿 API와 유사하지만 논블럭킹에 호환이 가능한 코어 웹핸들러 API 포함하여 스프링 웹 플럭스의 기반이 되는 리액티브 기본이 포함되어 있습니다.

이 토대로 스프링 웹플럭스는 두 가지 프로그래밍 모델을 선택 할 수 있습니다.

- Annotated Controllers: 스프링 MVC, 같은 어노테이션에 근간을 둔 스프링 웹 모듈과 관련이 있습니다. 스프링 MVC 그리고 웹플럭스 컨트롤러는 리액티브(Reactor 그리고 RxJava)의 리턴 타입을 제공하고, 결과적으로 이것을 구별하는 것이 쉽지 않습니다. 한 가지 주목할 만한 차이점은 웹플럭스가 리액티브 @RequestBody 아규먼트를 지원하는 것입니다.

- Functional Endpoints: 람다에 근간하고, 가볍고 그리고 함수형 프로그래밍 모델입니다. 어플리케이션이 요청을 라우팅하고 처리하는 데 사용 할 수 있는 작은 라이브러리 또는 유틸리티 집합이라고 생각할 수 있습니다. 어노테이션 컨트롤러와 큰 차이점 은 어노테이션을 통해 의도를 선언하고 콜백 되는것과 비교하여 처음 부터 끝까지 요청 처리를 담당한다는 것입니다.


### 1.1.4 적용 가능성

스프링 MVC 또는 웹플럭스?

당연한 질문이지만 불건전한 이분법을 설정하는 질문입니다. 실제로, 두 가지 모두 함께 작동하여 사용 가능한 옵션의 범위를 확장합니다. 이 둘은 서로 연속성과 일관성을 위해 설계되었으며, 나란히 사용할 수 있으며, 양측의 피드백은 양측 모두에게 도움이 됩니다. 다음 다이어그램은 두 개의 관계가 어떻게 연결되는지, 공통점 및 각각이 고유하게 지원하는 내용을 보여줍니다.

![](spring-mvc-and-webflux-venn.png)

다음과 같은 구체적인 사항을 고려해 보는 것이 좋습니다.

- 만약 잘 작동하는 스프링 MVC 어플리케이션이 있는 경우 변경할 필요가 없습니다. 명령형 프로그래밍은 코드를 작성하고, 이해하고, 디버그하는 가장 쉬운 방법입니다. 역사적으로 대부분의 라이브러리가 블락킹이므로 라이브러를 최대한 선택할 수 있습니다.

- 만약 논 블록킹 웹 스택을 선택하였다면, 스프링 웹플럭스는 다이어그램의 공간에 다른것과 동일한 실행 모델 이점을 제공하며 서버(Netty, Tomcat, Jetty, Underow 및 Servlet 3.1+ 컨테이너), 프로그래밍 모델 선택(어노테이션 컨트롤러 및 기능적인 웹 엔드포인트) 및 리액티브 라이브러리(Reactor, RxJava 또는 기타)를 선택할 수 있습니다.

- 만약 Java8 람다 또는 코틀린과 함께 사용할 수 있는 가볍고 기능적인 웹 프레임워크에 관심이 있는 경우 스프링 웹플럭스 함수형 웹 엔드포인트를 사용 할 수 있습니다. 그것은 더 큰 투명성과 컨트롤 이익을 얻을 수 있는 덜 복잡한 요구 사항을 가진 소규모 어플리케이션이나 마이크로 서비스에 좋은 선택이 될 것입니다.

- 마이크로서비스 아키텍처에서는 Spring MVC 또는 Spring WebFlux 컨트롤러 또는 Spring WebFlux 펑서널 엔드포인트와 함께 애플리케이션을 혼합할 수 있습니다. 두 프레임워크에서 동일한 어노테이션 기반 프로그래밍 모델을 지원하므로 올바른 작업에 적합한 도구를 선택하는 동시에 지식을 재사용하는 것이 더 쉬워집니다.

- 응용 프로그램을 평가하는 간단한 방법은 응용 프로그램의 디펜던시들을 확인하는 것입니다. 사용할 블록킹 퍼시스턴스 API(JPA, JDBC) 또는 네트워킹 API가 있는 경우, 적어도 일반적인 아키텍처에는 Spring MVC가 가장 적합합니다. Reactor와 RxJava 모두 별도의 스레드에서 차단 호출을 수행하는 것은 기술적으로 가능하지만 논블록킹 웹 스택을 최대한 활용하지는 못할 것입니다.

- 원격 서비스에 대한 호출이 있는 Spring MVC 응용 프로그램이 있는 경우 리액티브 WebClient를 사용해 보십시오. Spring MVC 컨트롤러 메서드에서 직접 리액티브 타입(Reactor, RxJava 또는 기타)을 반환할 수 있습니다. 요청당 대기 시간이 길거나 요청간 간의 상호 의존성이 클수록 이점이 더욱 커집니다. 스프링 MVC 컨트롤러는 다른 리액티브 구성 요소도 호출할 수 있습니다.

- 대규모 팀이 있다면 논블로킹, 펑셔널, 선언적 프로그래밍으로의 전환에서 가파른 러닝 커브를 염두에 두십시오. 전체 스위치 없이 시작하는 실용적인 방법은 리액티브 WebClient를 사용하는 것입니다. 그 외에도 작게 시작하여 이점을 측정합니다. 우리는 광범위한 어플리케이션 분야에서 (역주: 기술적)이동이 불필요할 것으로 예상합니다. 어떤 이점을 찾아야 할지 잘 모르겠으면 우선 논블록킹 I/O 작동 방식(예: 단일 스레드 Node.js의 동시성)과 그 효과에 대해 알아보십시오.

### 1.1.5 Servers

스프링 웹플럭스는 톰캣, 제티, 서블릿 3.1+ 컨테이너뿐만 아니라 넷티, 언더로우와 같은 서블릿이 아닌 런타임에서도 지원됩니다. 모든 서버는 low-level의 공통 API로 조정되어 higher-level 프로그래밍 모델이 여러 서버에서 지원될 수 있습니다.

Spring WebFlux는 서버를 시작하거나 중지할 수 있는 기본 제공 지원이 없습니다. 그러나 Spring 구성 및 WebFlux 인프라에서 응용 프로그램을 조립하고 몇 줄의 코드로 실행하는 것은 쉽습니다.

스프링 부트에는 이러한 단계를 자동화하는 WebFlux 스타터 기능이 있습니다. 기본적으로 스타터에서는 Netty를 사용하지만 Maven 또는 Gradle 종속성을 변경하여 Tomcat, Jetty 또는 Underow로 쉽게 전환할 수 있습니다. 스프링 부트는 비동기 논블록킹에서 더 널리 사용되고 클라이언트와 서버가 리소스를 공유할 수 있기 때문에 기본적으로 Netty로 설정됩니다.

Tomcat 및 Jetty는 Spring MVC 및 WebFlux와 함께 사용할 수 있습니다. 그러나, 그것들이 사용되는 방법은 매우 다르다는 것을 명심하세요. Spring MVC는 서블릿 블록킹 I/O에 의존하며 애플리케이션이 필요할 경우 서블릿 API를 직접 사용할 수 있도록 합니다. 스프링 웹플럭스는 서블릿 3.1 논블로킹 I/O에 의존하며, 로우 레벨 어댑터 뒤에서 서블릿 API를 사용한다. 직접 사용하기 위해 노출되지 않습니다.

Undertow의 경우 Spring WebFlux는 Servlet API없이 Undertow API를 직접 사용합니다.


### 1.1.6 Performance

퍼포먼스는 많은 특징과 의미를 가지고 있습니다. 일반적으로 리액티브 및 논블록킹은 애플리케이션 실행 속도를 향상 시키지 않습니다. 예를 들어  WebClient를 사용하여 원격 호출을 병렬로 실행하는 경우처럼 이러한 작업이 가능합니다. 전반적으로, 논블록킹 방식으로 일을 하기 위해서는 더 많은 작업이 필요하며, 이는 필요한 처리 시간을 약간 증가시킬 수 있습니다.

리액티브 및 논블록킹 기능의 주요 이점은 적은 수의 고정 스레드 및 적은 메모리로 확장할 수 있다는 것입니다. 따라서 예측 가능한 방식으로 확장되기 때문에 로드 시 애플리케이션의 복원력이 향상됩니다. 그러나 이러한 이점을 확인하려면 지연 시간(느린 네트워크 I/O와 예측 불가능한 네트워크 I/O의 혼합 포함)이 필요합니다. 여기서 리액티브 스택이 강점을 보이기 시작하고, 그 차이는 극적일 수 있습니다.

### 1.1.7 동시성 모델

스프링 MVC와 스프링 WebFlux는 모두 어노테이션 컨트롤러를 지원하지만 동시성 모델과 블락킹 및 스레드에 대한 기본 가정에는 중요한 차이가 있습니다.

스프링 MVC(및 일반적으로 서블릿 애플리케이션)에서는 애플리케이션이 현재 스레드를 블락 할 수 있다고 가정합니다(예: 원격 호출). 이러한 이유로 서블릿 컨테이너는 요청 처리 중에 잠재적인 블락킹을 처리하기 위해 큰 스레드 풀을 사용합니다.

Spring WebFlux(및 일반적으로 논블록킹 서버)에서는 응용 프로그램이 블록킹 되지 않는다고 가정합니다. 따라서 논블록킹 서버는 작은 고정 크기의 스레드 풀(이벤트 루프 작업자)을 사용하여 요청을 처리합니다.

> "스케일링"과 "스레드 수가 적음"은 모순되는 것처럼 들릴 수 있지만, 현재 스레드를 블록킹 하지 않는 것(대신 콜백에 의존함)은 처리할 블록킹 호출이 없기 때문에 추가 스레드가 필요하지 않다는 것을 의미합니다.

**블록킹 API 호출**

블록킹 라이브러리를 사용해야 하는 경우 어떻게 해야 합니까? Reactor와 RxJava는 모두 publishOn 연산자를 제공하여 다른 스레드에서 처리를 계속합니다. 그것은 쉬운 탈출구가 있다는 것을 의미합니다. 그러나 블록킹 API는 이 동시성 모델에 적합하지 않습니다.

**변경 가능 상태**

Reactor 및 RxJava에서는 연산자를 통해 논리를 선언합니다. 런타임에 반응형 파이프라인이 형성되며, 데이터는 서로 다른 단계에서 순차적으로 처리됩니다. 이것의 주요 이점은 파이프라인 내의 애플리케이션 코드가 동시에 호출되지 않기 때문에 애플리케이션이 변경 가능한 상태를 보호할 필요가 없다는 것입니다.

**쓰레드 모델**

Spring WebFlux로 실행되는 서버에서 어떤 스레드가 예상될까요?

- "vanilla" Spring WebFlux 서버 (예 : 데이터 액세스 나 다른 선택적 종속성 없음)에서 서버에 대해 하나의 스레드와 요청 처리 (일반적으로 CPU 코어 수만큼 많음)를 기대할 수 있습니다. 그러나 서블릿 컨테이너는 서블릿 (블록킹) I/O 및 서블릿 3.1 (논블록킹) I/O 사용을 모두 지원하는 더 많은 스레드 (예 : Tomcat의 10)로 시작할 수 있습니다.

- 리액티브 WebClient는 이벤트 루프 스타일로 작동합니다. 따라서 이와 관련된 처리 스레드의 수가 일정하지 않은 것을 볼 수 있습니다(예: Reactor Netty 커넥터가 있는 reactor-http-nio-). 그러나 리액터 Netty가 클라이언트와 서버 모두에 사용되는 경우 기본적으로 두 서버는 이벤트 루프 리소스를 공유합니다.

- Reactor 및 RxJava는 다른 스레드 풀로 처리를 전환하는 데 사용되는 publishOn 연산자와 함께 사용할 스레드 풀 추상화(스케줄러)를 제공합니다. 스케줄러는 "parallel"(스레드 수가 제한된 CPU 바인딩 작업의 경우) 또는 "elastic"(스레드 수가 많은 I/O 바인딩 작업의 경우)과 같은 특정 동시성 전략을 제안하는 이름을 가지고 있습니다. 이러한 스레드가 표시되면 일부 코드가 특정 스레드 풀 스케줄러 전략을 사용하고 있음을 의미합니다.

- 데이터 액세스 라이브러리 및 기타 타사 종속성 또한 자체 스레드를 생성하고 사용할 수 있습니다.

**Configuring**

Spring Framework는 서버 시작 및 중지를 지원하지 않습니다. 서버에 대한 스레딩 모델을 구성하려면 서버별 구성 API를 사용하거나, 스프링 부트를 사용하는 경우 각 서버에 대한 스프링 부트 구성 옵션을 확인해야 합니다. Web Client를 직접 구성할 수 있습니다. 다른 모든 라이브러리는 해당 설명서를 참조하십시오.

