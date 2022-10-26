# 1.3 디스패처 핸들러

Spring MVC와 유사한 Spring WebFlux는 중앙 WebHandler인 DispatcherHandler가 요청 처리를 위한 공유 알고리즘을 제공하는 반면 실제 작업은 구성 가능한 델리게이트 구성 요소에 의해 수행되는 프론트 컨트롤러 패턴을 중심으로 설계되었습니다. 이 모델은 유연하고 다양한 워크플로를 지원합니다.

DispatcherHandler는 Spring 구성에서 필요한 대리자 구성 요소를 찾습니다. 또한 Spring 빈 자체가 되도록 설계되었으며 실행되는 컨텍스트에 액세스하기 위해 ApplicationContextAware를 구현합니다. WebHandler라는 빈 이름으로 DispatcherHandler를 선언하면 WebHandler API에 설명된 대로 요청 처리 체인을 구성하는 WebHttpHandlerBuilder에 의해 차례로 발견됩니다.

WebFlux 응용 프로그램의 스프링 구성에는 일반적으로 다음이 포함됩니다.

- Bean 이름이 webHandler인 DispatcherHandler

- WebFilter 및 WebExceptionHandler 빈

- DispatcherHandler 특별한 빈

- 그 외

다음 예제에서 볼 수 있듯이 WebHttpHandlerBuilder에 프로세싱 체인을 구축하기 위한 구성이 제공됩니다.

```java
ApplicationContext context = ...
HttpHandler handler = WebHttpHandlerBuilder.applicationContext(context).build();
```

결과로 HttpHandler는 서버 어댑터와 함께 사용할 준비가 되었습니다.

## 1.3.1 특수 빈(Speicail Beans) 타입들

DispatcherHandler는 요청을 처리하고 적절한 응답을 렌더링하기 위해 특수 빈에 위임합니다. "특수 빈"이란 WebFlux 프레임워크 계약을 구현하는 Spring 관리 개체 인스턴스를 의미합니다. 일반적으로 기본 제공 계약과 함께 제공되지만 속성을 사용자 지정하거나 확장하거나 바꿀 수 있습니다.

다음 표는 DispatcherHandler에서 감지한 특수 Bean을 나열합니다. 낮은 수준에서 감지된 다른 빈도 있다는 점에 유의하십시오(웹 핸들러 API의 특수 빈 유형 참조).

|빈타입|설명|
|------|---|
|HandlerMapping|요청을 처리기에 매핑합니다. 매핑은 몇 가지 기준을 기반으로 하며 세부 사항은 HandlerMapping 구현 (어노테이션 컨트롤러, 간단한 URL 패턴 매핑 등)에 따라 다릅니다. 주요 HandlerMapping 구현은 @RequestMapping 주석 메서드를 위한 RequestMappingHandlerMapping, 기능적 엔드포인트 경로를 위한 RouterFunctionMapping, URI 경로 패턴 및 WebHandler 인스턴스의 명시적 등록을 위한 SimpleUrlHandlerMapping입니다.|
|HandlerAdapter|핸들러가 실제로 호출되는 방식에 관계없이 요청에 매핑된 핸들러를 호출하도록 DispatcherHandler를 돕습니다. 예를 들어 어노테이션이 달린 컨트롤러를 호출하려면 어노테이션을 해결해야 합니다. HandlerAdapter의 주요 목적은 이러한 세부 사항으로부터 DispatcherHandler를 보호하는 것입니다.|
|HandlerResultHandler|핸들러 호출의 결과를 처리하고 응답을 완료합니다. 결과 처리를 참조하십시오.|

## 1.3.2. 웹플럭스 설정

애플리케이션은 요청을 처리하는 데 필요한 인프라 빈(Web Handler API 및 DispatcherHandler 아래에 나열됨)을 선언할 수 있습니다. 그러나 대부분의 경우 WebFlux Config가 가장 좋은 시작점입니다. 필요한 Bean을 선언하고 이를 사용자 정의하기 위한 상위 레벨 구성 콜백 API를 제공합니다.

> Spring Boot는 WebFlux 구성에 의존하여 Spring WebFlux를 구성하고 많은 추가 편리한 옵션을 제공합니다.

## 1.3.3. Processing

DispatcherHandler는 다음과 같이 요청을 처리합니다.

- 각 HandlerMapping은 일치하는 핸들러를 찾도록 요청받고 첫 번째 일치가 사용됩니다.

- 핸들러가 발견되면 적절한 HandlerAdapter를 통해 실행되며 실행에서 반환 값을 HandlerResult로 노출합니다.

- HandlerResult는 응답에 직접 작성하거나 뷰를 사용하여 렌더링하여 처리를 완료하기 위해 적절한 HandlerResultHandler에 제공됩니다.

## 1.3.4 결과 핸들링

HandlerAdapter를 통한 핸들러 호출의 반환 값은 일부 추가 컨텍스트와 함께 HandlerResult로 래핑되고 이에 대한 지원을 요청하는 첫 번째 HandlerResultHandler에 전달됩니다. 다음 표는 WebFlux 구성에서 선언된 사용 가능한 HandlerResultHandler 구현을 보여줍니다.

|핸들러 타입 결과|리턴 값|디폴트 Order|
|------|---|---|
|ResponseEntityResultHandler|@Controller 인스턴스로의 전형적인 ResponseEntity|0|
|ServerResponseResultHandler|펑셔널 엔드포인트의 전형적인 ServerResponse|0|
|ResponseBodyResultHandler|@ResponseBody 메서드나 @RestController 클래스들의 응닶 값을 다룹니다.|100|
|ViewResolutionResultHandler|CharSequence, View, Model, Map, Rendering 또는 기타 객체는 모델 속성으로 처리됩니다. View Resolution을 참조하십시오.|Integer.MAX_VALUE|

## 1.3.5. 예외

HandlerAdapter에서 반환된 HandlerResult는 일부 처리기별 메커니즘을 기반으로 하는 오류 처리를 위한 함수를 노출할 수 있습니다. 이 오류 함수는 다음과 같은 경우에 호출됩니다.

- 핸들러 (예: @Controller) 호출이 실패

- HandlerResultHandler를 통한 핸들러 반환 값의 처리가 실패

오류 함수는 핸들러에서 반환된 반응 유형이 데이터 항목을 생성하기 전에 오류 신호가 발생하는 한 응답(예: 오류 상태)을 변경할 수 있습니다.

이것이 @Controller 클래스의 @ExceptionHandler 메소드가 지원되는 방식입니다. 대조적으로 Spring MVC에서 동일한 지원은 HandlerExceptionResolver에 구축됩니다. 이것은 일반적으로 중요하지 않습니다. 그러나 WebFlux에서는 @ControllerAdvice를 사용하여 핸들러가 선택되기 전에 발생하는 예외를 처리할 수 없음을 명심하십시오.

"어노테이션이 있는 컨트롤러" 섹션의 예외 관리 또는 WebHandler API 섹션의 예외도 참조하십시오.


### 1.3.6. View Resolution

뷰 레졸루션을 사용하면 특정 보기 기술에 얽매이지 않고 HTML 템플릿과 모델을 사용하여 브라우저에 렌더링할 수 있습니다. Spring WebFlux에서 View Resolver 인스턴스를 사용하여 String(논리적 보기 이름을 나타냄)을 View 인스턴스에 매핑하는 전용 HandlerResultHandler를 통해 보기 확인이 지원됩니다. 그런 다음 보기를 사용하여 응답을 렌더링합니다.

### 핸들링

ViewResolutionResultHandler에 전달된 HandlerResult에는 핸들러의 반환 값과 요청 처리 중에 추가된 속성이 포함된 모델이 포함됩니다. 반환 값은 다음 중 하나로 처리됩니다.

- String, CharSequence: 구성된 ViewResolver 구현 목록을 통해 View로 해석될 논리적 뷰 이름.

- void: 선행 및 후행 슬래시를 뺀 요청 경로를 기반으로 기본 보기 이름을 선택하고 이를 보기로 확인합니다. 뷰 이름이 제공되지 않았거나(예: 모델 속성이 반환됨) 비동기 반환 값(예: Mono가 비어 있음)이 제공되지 않은 경우에도 마찬가지입니다.

- 렌더링: 뷰 레졸루션 시나리오용 API입니다. 코드 완성 기능으로 IDE의 옵션을 살펴보세요.

- 모델, 맵: 요청을 위해 모델에 추가할 추가 모델 속성입니다.

- 기타: 다른 모든 반환 값(BeanUtils#isSimpleProperty에 의해 결정된 단순 유형 제외)은 모델에 추가될 모델 속성으로 처리됩니다. 속성 이름은 처리기 메서드 @ModelAttribute 주석이 없는 경우 규칙을 사용하여 클래스 이름에서 파생됩니다.

모델은 비동기, 리액티브 타입(예: Reactor 또는 RxJava에서)을 포함할 수 있습니다. 렌더링하기 전에 AbstractView는 이러한 모델 속성을 구체적인 값으로 해석하고 모델을 업데이트합니다. 단일 값(싱글 밸류) 리액티브 타입은 단일 값 또는 값 없음(비어 있는 경우)으로 확인되는 반면 다중 값 리액티브 타입(예: Flux<T>)은 수집되어 List<T>로 확인됩니다.

뷰 레졸루션을 구성하는 것은 Spring 구성에 ViewResolutionResultHandler 빈을 추가하는 것만 큼 간단합니다. WebFlux Config는 보기 확인을 위한 전용 구성 API를 제공합니다.

Spring WebFlux와 통합된 보기 기술에 대한 자세한 내용은 보기 기술을 참조하십시오.\


### 리다이렉팅

보기 이름에 특수 redirect: 접두사를 사용하면 리디렉션을 수행할 수 있습니다. UrlBasedViewResolver(및 하위 클래스)는 이를 리디렉션이 필요하다는 명령으로 인식합니다. 나머지 보기 이름은 리디렉션 URL입니다.

순 효과는 컨트롤러가 RedirectView 또는 Rendering.redirectTo("abc").build()를 반환한 것과 같지만 이제 컨트롤러 자체가 논리적 뷰 이름으로 작동할 수 있습니다. redirect:/some/resource와 같은 보기 이름은 현재 애플리케이션에 상대적인 반면, redirect:https://example.com/arbitrary/path와 같은 보기 이름은 절대 URL로 리디렉션됩니다.

### 콘텐츠 협상

ViewResolutionResultHandler는 콘텐츠 협상을 지원합니다. 요청 미디어 유형을 선택한 각 보기에서 지원하는 미디어 유형과 비교합니다. 요청된 미디어 유형을 지원하는 첫 번째 보기가 사용됩니다.

Spring WebFlux는 JSON 및 XML과 같은 미디어 유형을 지원하기 위해 HttpMessageWriter를 통해 렌더링되는 특수 View인 HttpMessageWriterView를 제공합니다. 일반적으로 WebFlux 구성을 통해 이를 기본 보기로 구성합니다. 기본 보기는 요청된 미디어 유형과 일치하는 경우 항상 선택되고 사용됩니다.
