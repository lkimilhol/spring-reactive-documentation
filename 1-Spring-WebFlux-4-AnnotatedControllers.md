# 1.4 어노테이티드 컨트롤러

Sprjava
ing WebFlux는 @Controller 및 @RestController 구성
요소가 어노테이션을 사용하여 요청 매핑, 요청 입력, 예외 처리 등을 표현하는 어노테이션 기반 프로그래밍 모델을 제공합니다. 어노테이션이 달린 컨트롤러는 유연한 메서드 시그네춰를 가지고 있으며 기본 클래스를 확장하거나 특정 인터페이스를 구현할 필요가 없습니다.

다음 목록은 기본 예를 보여줍니다.

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String handle() {
        return "Hello WebFlux";
    }
}
```

위의 예에서 메소드는 응답 바디에 쓸 문자열을 반환합니다.

## 1.4.1. @Controller

표준 Spring 빈 정의를 사용하여 컨트롤러 빈을 정의할 수 있습니다. @Controller 스테레오타입은 자동 감지를 허용하고 클래스 경로에서 @Component 클래스를 감지하고 이에 대한 빈 정의를 자동 등록하기 위한 Spring 일반 지원과 정렬됩니다. 또한 주석이 달린 클래스에 대한 스테레오타입 역할을 하여 웹 구성 요소로서의 역할을 나타냅니다.

이러한 @Controller Bean의 자동 탐지를 활성화하려면 다음 예시와 같이 Java 구성에 구성 요소 검색을 추가할 수 있습니다.

```java
@Configuration
@ComponentScan("org.example.web") 
public class WebConfig {

    // ...
}
```

@RestController는 자체적으로 @Controller 및 @ResponseBody로 메타 어노테이션이 달린 합성된 애노테이션으로, 모든 메소드가 top-level @ResponseBody 어노테이션을 상속하므로 응답 바디 대 뷰 레졸루션 및 HTML 템플릿 렌더링을 통해 직접 작성하는 컨트롤러를 나타냅니다.

## 1.4.2. Request Mapping

@RequestMapping 어노테이션은 요청을 컨트롤러 메서드에 매핑하는 데 사용됩니다. URL, HTTP 메서드, 요청 매개 변수, 헤더 및 미디어 타입별로 일치시킬 수 있는 다양한 속성이 있습니다. 클래스 수준에서 공유 매핑을 표현하거나 메서드 수준에서 사용하여 특정 엔드포인트 매핑으로 범위를 좁힐 수 있습니다.

@RequestMapping의 HTTP 메서드별 바로 적용 가능한 어노텡션도 있습니다.

- @GetMapping

- @PostMapping

- @PutMapping

- @DeleteMapping

- @PatchMapping

위의 어노테이션은 대부분의 컨트롤러 메서드가 특정 HTTP 메서드에 매핑되어야 하기 때문에 제공되는 사용자 지정 주석입니다. @RequestMapping은 기본적으로 모든 HTTP 메서드와 일치합니다. 동시에 공유 매핑을 표현하기 위해서는 클래스 레벨에서 @RequestMapping이 여전히 필요합니다.

다음 예제에서는 유형 및 메서드 수준 매핑을 사용합니다.

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

### URI Patterns

글로벌 패턴과 와일드카드를 사용하여 요청을 매핑할 수 있습니다.

| 패턴             |설명| 예시                                                                                                                                                               |
|----------------|---|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ?              |한 문자와 일치합니다.| "/pages/t?st.html" 는 "/pages/test.html" 와 매치합니다. 또 "/pages/t3st.html"                                                                                            |
| *              |경로 세그먼트 내에서 0개 이상의 문자를 일치시킵니다.| "/resources/\*.png" 와 "/resources/file.png" 매치합니다."/projects/*/versions" 와 "/projects/spring/versions" 가 매치합니다. 그러나 "/projects/spring/boot/versions"와 일치하지 않습니다. |
| **             |경로의 끝까지 0개 이상의 경로 세그먼트를 일치시킵니다.| "/resources/\**" 와 "/resources/file.png" 가 일치합니다. 그리고 "/resources/images/file.png""/resources/**/file.png" 는 유효하지 않습니다. **는 경로 끝에만 허가 됩니다..                      |
| {name}         |경로 세그먼트를 일치시키고 "name"이라는 변수로 캡처합니다.| "/projects/{project}/versions" 와 "/projects/spring/versions"매치합니다. 그리고 project=spring을 감지합니다.                                                                    |
| {name:[a-z]+}  |regexp "[a-z]+"를 "name"이라는 경로 변수로 일치시킵니다.| "/projects/{project:[a-z]+}/versions" 와 "/projects/spring/versions" 마치합니다. 하지만 "/projects/spring1/versions" 은 매치되지 않습니다.                                         |
| {*path}        |경로가 끝날 때까지 0개 이상의 경로 세그먼트를 일치시키고 "경로"라는 변수로 캡처합니다.| "/resources/{*file}" 와 "/resources/images/file.png" 매치합니다. 그리고 file=/images/file.png 를 캡처합니다.                                                                    |

캡처된 URI 변수는 다음 예시와 같이 @PathVariable을 사용하여 액세스할 수 있습니다.

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

다음 예시와 같이 클래스 및 메서드 수준에서 URI 변수를 선언할 수 있습니다.

```java
@Controller
@RequestMapping("/owners/{ownerId}") 
public class OwnerController {

    @GetMapping("/pets/{petId}") 
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```
URI 변수가 자동으로 적절한 형식으로 변환되거나 TypeMismatchException이 발생합니다. 기본적으로 간단한 유형(int, long, Date 등)이 지원되며 다른 데이터 유형에 대한 지원을 등록할 수 있습니다. 유형 변환 및 데이터 바인더를 참조하십시오.

URI 변수의 이름은 명시적으로 지정할 수 있지만(예: @PathVariable("customId") 이름이 동일하고 디버깅 정보 또는 Java 8의 -parameters 컴파일러 플래그를 사용하여 코드를 컴파일할 경우 해당 세부 정보를 생략할 수 있습니다.

{*varName} 구문은 0개 이상의 나머지 경로 세그먼트와 일치하는 URI 변수를 선언합니다. 예를 들어 /resources/{*path}은(는) /resources/ 아래의 모든 파일과 일치하며 "path" 변수는 /resources 아래의 전체 경로를 캡처합니다.

{varName:regex} 구문이 {varName:regex}인 정규식으로 URI 변수를 선언합니다. 예를 들어 /spring-web-3.0.5.jar의 URL이 주어지면 다음 메서드는 이름, 버전 및 파일 확장자를 추출합니다.

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI 경로 패턴에는 로컬, 시스템, 환경 및 기타 속성 소스에 대해 PropertySourcesPlaceholderConfigurer를 통해 시작 시 해결되는 ${…}개의 자리 표시자가 포함될 수도 있습니다. 이를 통해 예를 들어 일부 외부 구성을 기반으로 기본 URL을 매개 변수화할 수 있습니다.

> Spring WebFlux는 URI 경로 일치 지원을 위해 PathPattern 및 PathPatternParser를 사용합니다. 두 클래스 모두 스프링 웹에 위치하며 런타임에 다수의 URI 경로 패턴이 일치하는 웹 응용 프로그램에서 HTTP URL 경로와 함께 사용하도록 명시적으로 설계되었습니다.

Spring WebFlux는 /person.*과 같은 매핑이 /person.*과도 일치하는 Spring MVC와 달리 접미사 패턴 일치를 지원하지 않습니다. URL 기반 콘텐츠 협상의 경우 필요한 경우 쿼리 매개 변수를 사용하는 것이 좋습니다. 이 매개 변수는 더 간단하고, 더 명확하며, URL 경로 기반 공격에 덜 취약합니다.

### Pattern Comparison

여러 패턴이 URL과 일치하면 가장 적합한 패턴을 찾기 위해 비교해야 합니다. 이것은 좀 더 구체적인 패턴을 찾는 PathPattern.SPECIFICITIY_COMPARATOR로 수행됩니다.

모든 패턴에 대해 URI 변수와 와일드 카드의 수에 따라 점수가 계산됩니다. 여기서 URI 변수 점수는 와일드 카드보다 낮습니다. 총점이 낮은 패턴이 승리하며, 두 패턴이 같은 점수일 경우 더 긴 것을 선택합니다. 

캐치 올 패턴 (예 : **, {*varName})은 득점에서 제외되며 항상 마지막으로 정렬됩니다. 두 패턴이 모두 캐치 올이면 더 긴 패턴이 선택됩니다.

### Consumable Media Types

다음 예시와 같이 요청의 내용 유형을 기준으로 요청 매핑을 좁힐 수 있습니다.

```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet) {
    // ...
}
```
consumes 속성은 부정 표현식도 지원합니다. 예를 들어 !text/plain은 텍스트/plain 이외의 모든 내용 유형을 의미합니다.클래스 수준에서 공유 소비 속성을 선언할 수 있습니다. 그러나 대부분의 다른 요청 매핑 속성과 달리 클래스 수준에서 사용될 때 메서드 수준은 클래스 수준 선언을 확장하지 않고 속성 재정의를 사용합니다.

> MediaType은 일반적으로 사용되는 미디어 유형 | 예를 들어 Application_JSON_VALUE 및 Application_XML_VALUE에 대한 상수를 제공합니다.

### Producible Media Types

다음 예제에서 볼 수 있듯이 Accept request 헤더와 컨트롤러 메소드가 생성하는 콘텐츠 유형 목록을 기반으로 요청 매핑을 좁힐 수 있습니다.

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json")
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

미디어 유형은 문자 집합을 지정할 수 있습니다. 부정 표현식은 지원됩니다. 예를 들어 !text/plain은 텍스트/일반 이외의 모든 콘텐츠 유형을 의미합니다.

> MediaType은 일반적으로 사용되는 미디어 유형에 대해 상수를 제공합니다(예: APPLICATION_JSON_VALUE, APPLICATION_XML_VALUE).

### Parameters and Headers

쿼리 매개 변수 조건을 기준으로 요청 매핑을 좁힐 수 있습니다. 쿼리 매개 변수 (myParam)의 존재 여부, 부재 (! myParam) 또는 특정 값 (myParam = myValue)을 테스트 할 수 있습니다. 다음 예제는 값이 있는 매개 변수를 테스트합니다.

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") // Check that myParam equals myValue.

public void findPet(@PathVariable String petId) {
    // ...
}
```

다음 예시와 같이 요청 헤더 조건과 동일한 조건을 사용할 수도 있습니다.

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") // Check that myHeader equals myValue
public void findPet(@PathVariable String petId) {
    // ...
}
```

### HTTP HEAD, OPTIONS

@GetMapping 및 @RequestMapping(메서드=Method)입니다.GET)는 요청 매핑을 위해 HTTP HEAD를 명백하게 지원합니다. 컨트롤러 메서드는 변경할 필요가 없습니다. HttpHandler 서버 어댑터에 적용된 응답 래퍼는 Content-Length 헤더가 응답에 실제로 쓰지 않고 쓰여진 바이트 수로 설정되도록 합니다.

기본적으로 HTTP OPTIONS는 URL 패턴이 일치하는 모든 @RequestMapping 메서드에 나열된 HTTP 메서드 목록으로 응답 허용 헤더를 설정하여 처리됩니다.

HTTP 메소드 선언이 없는 @RequestMapping의 경우 허용 헤더는 GET, HEAD, POST, PUT, PATCH, DELET, OPTIONS로 설정됩니다. 컨트롤러 메소드는 항상 지원되는 HTTP 메소드를 선언해야합니다 (예 : HTTP 메소드 특정 법규-  @GetMapping, @PostMapping 등).

@RequestMapping 메서드를 HTTP HEAD 및 HTTP OPTIONS에 명시적으로 매핑할 수 있지만 일반적인 경우에는 그럴 필요가 없습니다.

### Custom Annotations

Spring WebFlux는 요청 매핑을 위해 구성된 어노테이션의 사용을 지원합니다. 이러한 어노테이션들은 그 자체가 @RequestMapping으로 메타 어노테이션을 달고 더 좁고 구체적인 목적을 가진 @RequestMapping 속성의 하위 집합(또는 모든)을 다시 선언하도록 구성된 어노테이션입니다.

@GetMapping, @PostMapping, @PutMapping, @DeleteMapping 및 @PatchMapping은 구성된 어노테이션의 예입니다. 대부분의 컨트롤러 메서드는 특정 HTTP 메서드에 매핑되어야 하며 @RequestMapping은 기본적으로 모든 HTTP 메서드와 일치하기 때문에 제공됩니다. 구성된 어노테이션의 예가 필요한 경우, 어노테이션의 선언 방식을 살펴보십시오.

Spring WebFlux는 또한 사용자 지정 요청 일치 로직을 사용하여 사용자 지정 요청 매핑 속성을 지원합니다. 이는 RequestMappingHandlerMapping을 하위 클래스화하고 getCustomMethodCondition 메소드를 재정의해야 하는 고급 옵션으로 사용자 지정 속성을 확인하고 자신의 RequestCondition을 반환할 수 있습니다.

### 명시적 등록

동적 등록 또는 동일한 핸들러의 다른 인스턴스(예: 다른 URL)와 같은 고급 사례에 사용할 수 있는 핸들러 메서드를 프로그래밍 방식으로 등록할 수 있습니다. 다음 예제에서는 이 작업을 수행하는 방법을 보여 줍니다.

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }

}
```

## 1.4.3 Handler Methods

@RequestMapping 핸들러 메소드는 유연한 시그네춰를 가지고 있으며 지원되는 다양한 컨트롤러 메소드 아규먼트 및 반환 값 중에서 선택할 수 있습니다.

### Method Arguments

다음 표는 지원되는 컨트롤러 메서드 아규먼트를 보여줍니다. 리액티브 타입(Reactor, RxJava 또는 기타)은 블록킹 I/O(예: 요청 바디 읽기)을 지원하는 아규먼트에서 지원됩니다. 이 부분은 설명 열에 표시됩니다. 블록킹이 필요가 없는 인수에는 리액티브 타입이 필요하지 않습니다.

| 패턴                                    | 설명                                                                                              |
|---------------------------------------|-------------------------------------------------------------------------------------------------|
| ServerWebExchange                     | ServerWebExchange에 접근합니다- HTTP 요청과 응답, 요청과 세션 어트리뷰트, checkNotModified 메서드, 기타 등등을 위한 컨테이너 입니다.  |
| ServerHttpRequest, ServerHttpResponse | Http 요청 또는 응답에 접근합니다.                                                                           |
| WebSession                            | 세션에 접근합니다. 어트리뷰트가 추가되지 않는 한 새 세션을 강제로 시작하지 않습니다. 리액티브 타입을 지원합니다.                                |
| java.security.Principal               | 현재 인증된 사용자- 알려진 경우 특정 Principal 구현 클래스일 수 있습니다. 리액티브 타입을 지원합니다.                                 |
| org.springframework.http.HttpMethod   | 요청의 HTTP 메서드입니다.                                                                                |
| java.util.Locale                      | 현재 요청 로케일이, 가장 구체적인 사용가능한 로케일리졸버를 결정합니다. - 실제로 구성된 LocaleResolver/LocaleContextResolver가 사용됩니다. |
| java.util.TimeZone + java.time.ZoneId | LocaleContextResolver에 의해 결정된 현재 요청과 연관된 표준 시간대.                                                |
| @PathVariable                         | URI 템플릿 변수에 대한 액세스용입니다. URI 패턴을 참조하십시오.                                                         |
| @MatrixVariable                       | URI 경로 세그먼트의 이름-값 쌍에 대한 액세스용입니다. 자세한 내용은 메트릭스 변수를 참조하십시오.                                       |


