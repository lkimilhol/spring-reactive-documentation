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

