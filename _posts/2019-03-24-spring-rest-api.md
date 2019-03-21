---
layout: post
title: "스프링 기반 REST API 개발 (백기선님 인프런 강좌)"
author: academey
categories: Spring
---

진행중

# 1. REST API

API

- Application Programming Interface
  REST
- Representational State Transfer
- 인터넷 상의 시스템 간의 상호 운용성을 제공하는 방법중 하나
- 시스템 제각각의 독립적인 진화를 보장하기 위한 방법
- REST API : REST 아키텍쳐 스타일을 따르는 API

REST 아키텍쳐 스타일 (발표 영상 11분)

- HATEOS 를 만족하려면, 어플리케이션으로의 전이가 가능해야 한다.
  - 방법 1. data로
    - data에 다양한 방법으로 하이퍼링크를 표현한다.
    - Https://example.org/todos/{id} 도 가능
  - 방법 2. HTTP 헤더로
    - Link로 추가.
    - Link : </todos>; rel=“collection"
    - Location: /todos/1
- 서버의 기능이 변경되어도 클라이언트를 업데이트 할 필요가 없다.
  - 서버가 보낸 메시지만 봐도 클라이언트가 알아서 해석해서 동작할 수 있다는 말이다.
- 그래서 Uniform Interface 를 쭉 지켜줘야 한다.
- 항상 Self-descriptive 해야 한다. 어떤 문서 타입인지, 다른 전이가 가능하도록 링크가 있는지, id 가 뭘 의미하는지, title 이 뭘 의미하는지 모른다.
  - 방법 1. Media type 정의.
    - 미디어 타입을 정의한다. Application/vnd.todos+json
    - 미디어 타입 문서를 작성한다.
    - IANA에 가서 미디어 타입을 등록한다.
    - 이제 이 명세를 찾아갈 수 있으므로 메세지를 온전히 해석할 수 있다.
    - 그런데 넘 번거롭다.
  - 방법 2. Profile
    - Link에 의미를 정의한 명세를 작성한 docs 를 달아둔다.
- Uniform Interface (발표 영상 11분 40초)
  _ Identification of resources
  _ manipulation of resources through represenations
  _ self-descrive messages
  _ hypermisa as the engine of appliaction state (HATEOAS)
  실제로 지금 오픈되어 있는 많은 API들을 보면, self-descriptive 하지 않으며(무슨 의미인지 모름) 전이 불가능하다.
  깃헙의 api가 진짜다. 미디어 타입도 다 올려놓는다.

우리는 어떻게 할거냐.
Self-descriptivae message

- Profile 링크 헤더를 추가한다.
  HATEOS 해결 방법
- 데이터에 링크를 HAL(Hypertext Application Language) 로 제공한다.

#2. Event REST API
이벤트 등록, 조회 및 수정 API를 만들거다.
스프링 Docs 로 API 문서 찍어주고, 스프링 HATEOS 로 링크 찍어줄거다.

1.  이벤트 목록 조회 REST API (로그인 안 한 상태)
    GET /api/events
    ● 응답에 보여줘야 할 데이터
    ○ 이벤트 목록
    ○ 링크
    ■ self
    ■ profile: 이벤트 목록 조회 API 문서로 링크
    ■ get-an-event: 이벤트 하나 조회하는 API 링크
    ■ next: 다음 페이지 (optional)
    ■ prev: 이전 페이지 (optional)

        ● 문서?
            ○ 스프링 REST Docs로 만들 예정

2.  이벤트 목록 조회 REST API (로그인 한 상태)
    GET /api/events
    ● 응답에 보여줘야 할 데이터
    ○ 이벤트 목록
    ○ 링크
    ■ self
    ■ profile: 이벤트 목록 조회 API 문서로 링크
    ■ get-an-event: 이벤트 하나 조회하는 API 링크
    ■ create-new-event: 이벤트를 생성할 수 있는 API 링크
    ■ next: 다음 페이지 (optional)
    ■ prev: 이전 페이지 (optional)
    ● 로그인 한 상태???? (stateless라며..)
    ○ 아니, 사실은 Bearer 헤더에 유효한 AccessToken이 들어있는 경우!

3.  이벤트 생성
    POST /api/events

4.  이벤트 하나 조회
    GET /api/events/{id}

5.  이벤트 수정
    PUT /api/events/{id}

# 3. Events API 사용 예제 (PostMan & Restlet )

1. (토큰 없이) 이벤트 목록 조회

- create 안 보임

2. access token 발급 받기 (A 사용자 로그인)
3. (유효한 A 토큰 가지고) 이벤트 목록 조회

- create event 보임

4. (유효한 A 토큰 가지고) 이벤트 만들기
5. (토큰 없이) 이벤트 조회

- update 링크 안 보임

6. (유효한 A 토큰 가지고) 이벤트 조회

- update 링크 보임

7. access token 발급 받기 (B 사용자 로그인)
8. (유효한 B 토큰 가지고) 이벤트 조회

- update 안 보임

REST API 테스트 클라이언트 애플리케이션

- 크롬 플러그인
- Restlet

# 4. 프로젝트 만들기

자, 이제 시작하자.
추가할 의존성

- Web, JPA, HATEOAS, REST Docs, PostgreSQL, Lombok

도메인 클래스 만들기

- 이제 Event 라는 클래스를 만들자. 문서에 있으니 해당하는 것들 다 만들고, EventStatus 라는 Enum 클래스도 만들자. EventTest 를 만드는데, junit assert 가 기본으로 들어와있는데, 우리는 assertj 쓸 거니까 최적화 하자. (옵션 + 컨트롤 + O) 하면 위에 임포트 문 없어진다.
- 그리고 Event 에 @Builder 추가해서 빌더 패턴 쓰자. 롬복 빌더 패턴을 추가했다면, 커맨드 + , 눌러서 lombok plugin 을 설치하고, Enable annotation 을 해줘야 한다.
- 그런데 빌더 패턴만 추가했을 때, 기본적으로 빈이 생성될 때 생성자가 필요한데 빌더 패턴으로 생성된 건 모든 값이 있어야만 한다. 따라서 롬복 어노테이션 @AllArgsConstructor @NoArgsConstructor @Getter @Setter @EqualsAndHashCode(of=“id”) 들을 붙여주자. 그러면 빌더 패턴 외에 set 함수로도 사용 가능하다. 이 어노테이션을 줄일 수 있는 방법은 아직 없다.. 메타 어노테이션이라.
- @Data 를 쓰지 못하는 이유는, EqualsAndHashCode 를 지원하지만 모든 키 값을 가져가기 때문에 문제 생긴다. Entity 위에서는 쓰면 안 된다. // + 댓글을 보니 @Data @EqualsAndHashCode(of = “id”) 로 해결 가능하다고 한다.

# 5. Event 생성 API 구현 : 비즈니스 로직

Event 생성 API

- 다음의 입력 값을 받는다.
- name
- description
- beginEnrollmentDateTime
- closeEnrollmentDateTime
- beginEventDateTime
- endEventDateTime
- location (optional) 이게 없으면 온라인 모임
- basePrice (optional)
- maxPrice (optional)
- limitOfEnrollmentWeb, JPA, HATEOAS, REST Docs, PostgreSQL, Lombok

basePrice와 maxPrice 경우의 수와 각각의 로직’  
basePrice maxPrice
0 100 // 다 100원! 선착순 등록
0 0 무료
100 0 무제한 경매 (높은 금액 낸 사람이 등록)
100 200 제한가 선착순 등록 처음 부터 200을 낸 사람은 선 등록. 100을 내고 등록할 수 있으나 더 많이 낸 사람에 의해 밀려날 수 있음.

결과값

- id
- name
- ...
- eventStatus: DRAFT, PUBLISHED, ENROLLMENT_STARTED, ...
- offline
- free
- \_links
  - profile (for the self-descriptive message)
  - self
  - publish
  - ...

# 6. Event API 테스트 클래스 생성

스프링 부트 슬라이스 테스트

- TDD 를 해보자. EventControllerTests 클래스를 만들고, @RunWith(SpringRunner.class) @WebMvcTest 를 붙여서 가상 서블릿에다가 요청을 날릴 수 있도록 MockMvc를 주입받는다. 웹 관련 빈만 등록해준다. 슬라이싱 테스트를 할 수 있다. Dispatcher servlet, converter 등등을 이용.
- contentType(MediaType.APPLICATION_JSON_UTF8)로 보내고 .accept(MediaTypes.HAL_JSON)을 받는다.

# 7. Event API 구현 : 201 응답 받기

@RestController 만들기

- @Controller 를 붙이고 @PostMapping(“/api/events”) public ResponseEntity createEvent() { 메소드를 만들고 ResponseEntity를 리턴하자. 이 때, linktTO()와 methodOn() 을 사용하는 게 좋다.
- 공통으로 쓰는 path 인 api/events 는 위에 RequestMapping 에 붙이고, 내려줄 미디어 타입이 다 HAL이니까 그것도 써주자. 이제 path 따로 안 건들여도 되니까 뭐 methodOn? 은 안써도 된단다...

ResponseEntity 를 사용하는 이유

- 응답 코드, 헤더, 본문 모두 다루기 편한 API

Location API

- HATEOS가 제공하는 linkTo(), methodOn() 사용

객체를 JSON으로 변환

- ObjectMapper 사용

테스트 할 것

- 입력값들을 전달하면 JSON 응답으로 201 이 나오는지 확인
  - Location 헤더에 생서된 이벤트를 조회할 수 있는 URI가 담겨있는지 확인
  - Id 는 DB에 들어갈 때 자동 생성된 값으로 나오는지 확인

# 8. Event API 구현 : EventRepository 구현

우선, Event 를 Entity로 만들자. Id 에 @Id 붙이고, EventStatus에 @Enumerated(EnumType.STRING) 붙인다. 기본 값이 ORDINAL 인데 만약 순서 바뀌면 클나니까 그냥 스트링으로 하는 게 좋다.

이제 EventRepository 만들어서 JPARepository 상속받아서 구현,
그리고 Controller 에서 주입받은 다음에 테스트를 돌려보면 깨진다. EventRepository 가 없대. 왜지? 아, SliceTest 여서 웹 관련한 빈만 가져오지 다른 건 하나도 안 가져왔다. 따라서 EventRepository 를 목빈으로 등록해주자. 그런데 이렇게 목빈을 하면, 모든 메소드를 null 로 리턴하기 때문에 이것 또한 목킹을 해줘야 한다.
Mockito.when(eventRepository.save(event)).thenReturn(event);
요런 느낌으로.

그러면 응답하는 헤더도 HAL_JSON 으로 오고 있고, 201도 내려오고, event 객체도 id 딸려서 내려온다. 요것들도 다 테스트로 확인할 수 있다. header().exists(HttpHeaders.LOCATION) 뭐 이런식으로 다 테스트해보자.

# 9. Event API 구현 : 입력값 제한하기

이벤트를 입력할 때, id 나 free 혹은 offline 같은 값 들은 입력되면 안 된다.
자동 생성되거나, 가격이 없으면 계산이 되어 박히고, 위치가 없으면 자동으로 박혀야 하기 때문이다.

그래서 에노테이션을 추가해줘야 하는데, 지금 도메인에 추가하기에는 너무 부담스럽다.
너무 많아진다. 따라서, EventDTO 라는 입력 전용 클래스를 만들어서 빼두자. 입력받아야 하는 값들을 가지고 멤버 변수로 추가하자. (id, free, offline 제외한 모든 값)

그리고 @Data @Builder @NoArgsConstructor @AllArgsConstructor 들을 추가하자. 그리고 Controller 에서 파라미터로 받는 걸 EventDTO로 변경한다. 그러면 입력값에 필요한 것들만 받게 된다. 이제 EventDTO를 이벤트 타입으로 바꿔줘야 하는데, 이 때 쓸 수 있는 게 ModelMapper 라는 라이브러리가 있다. (DTO -> 도메인 객체로 값 복사)

의존성을 추가해주자. org.modelmapper::modelmapper 그리고 공용에서 쓸 거니까 빈으로 등록해주자. @Bean ModelMapper
그리고 컨트롤러에서 modelMapper.map(eventDto, Event.class); 로 받은 Event 객체를 저장한다. 이 때 테스트가 깨지는데, 왜냐면 이걸 다시 봐라.
Mockito.when(eventRepository.save(event)).thenReturn(event);
여기서 event 를 저장하면 event 를 리턴하는데, 우리가 저장한 event 는 위에서 설정했던 것과 다른 id 와 free 값을 가지고 있으므로 같은 event 가 아닌 걸로 인식 되어 event 를 리턴하지 않고 null을 리턴하기 때문이다.

따라서 mocking 을 하지 않겠다. Slice Test 로 만들지 말고, @SpringBootTests 로 변경하자. (SpringBootTests 에 디스패쳐 서블릿이 Mock 으로 들어가 있기 때문에 MockMvc 계속 사용이 가능하다. ) 그리고 MockMvc 빈을 가져오려면 @AutoConfigureMockMvc 를 넣어 주자.

그리고. MockBean EventRepository 를 없애주면, 실제 리포지토리에 넣어서 작동시켜 정상적인 값을 내려준다. 목킹 하는 게 너무 많으면 테스트가 불편하니까 이제 앞으로는 통합 테스트로 @SpringBootTests 를 써서 SpringBootApplication 아래에 있는 모든 빈을 등록해서 사용하자.

# 10. Event API 구현 : 입력값 이외의 에러 발생

위의 경우에는 그냥 DTO 에 안 박히는 걸로 해줬는데, 이제는 이상한 값이 오면 에러를 발생시켜보자. 원래 정상적인 값이 온 건 DTO로 변경하자. 그리고 못 넣는 값은 제거하자. 그리고 아래에 BadRequest 테스트를 만들고 이상한 값을 설정한 event 를 날려보자.

그런데 201을 내려준다. 이 경우에는 spring boot 가 제공하는 프로퍼티를 사용한 확장 오브젝터 기능을 사용하면 된다.

Deserialization : Json -> 오브젝트
Serialization : 오브젝트 -> Json 이라고 하는데, 우리는 요청 날릴 때 JSON 을 보내서 컨트롤러에서 오브젝트화 시키는 거니까 Deserialization을 하는 것이다.

프로퍼티에 spring.jackson.deserialization.fail-on-unknown-properties=true 조건을 주면, 모르는 프로퍼티가 오면 실패하도록 설정해주면 된다. 그러면 400이 내려간다. 경우에 따라 다른데, 엄격하게 내려주는 게 좋을 수도 있다. Jackson 오브젝트 매퍼를 설정할 수 있는 프로퍼티들이 많으니까 찾아보길 바란다.

# 11. Event API 구현 : Bad Request 처리하기

입력값이 이상한 경우, 즉 name 이 안 가는 경우들을 어떻게 처리할 것인가!

@Valid 와 BindingResult (또는 Errors)를 이용해서 처리하자.
컨트롤러에 @RequestBody @Valid EventDto 를 적으면 Entity에 바인딩 할 때 해당 객체를 검증할 수행할 수 있다. EventDto 에 @NotEmpty 등의 Validation 어노테이션을 추가한다. 그리고 해당 검증의 결과를 Errors에 넣어준다. Errors 는 Spring Validation 거 쓰자.
public ResponseEntity createEvent(@RequestBody @Valid EventDto eventDto, Errors errors) {

자 여기까지는 입력 데이터의 존재와 타입으로 에러를 찍어주는데, 만약 시작 날짜가 끝 날짜보다 늦고, basePrice 가 maxPrice 보다 크다면? 이런 것들은 annotation으로 검증할 수 없다. 따로 Validator 를 만들어줘야 한다.

@Component
Class EventValidator 를 만든 다음에, Public void validate(EventDto eventDto, Errors errors) 메소드를 구현하자. 그리고 eventDto 의 변수들을 조회하면서 errors를 추가해주면 되는데, errors.rejectValue() 에서 커맨드 + P를 누르면 내부 파라미터의 조건들을 써주니까 그거 조회하면 된다. 그렇게 validate 메소드를 만들었으면, Controller 에서 마찬가지로 errors 받아서 validate 조진다음에 errors 다시 조회해보면 된다.

테스트를 보기 좀 힘든데, 어노테이션을 만들어서 쓰자. Junit 에 Description 용으로 필요할 것 같다. Common package 를 만들고 TestDescription 을 만들자.
타겟은 메소드고, 언제까지 유지되느냐에 대한 조건을 달고 (우리는 컴파일 이후에 필요 없으니 소스로 쓰자) 어노테이션을 만들어서 테스트 위에 설명을 붙이자.
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface TestDescription {
String value();
}

뭐 이런 느낌이다. JUnit 5 로 올라가면 디스크립션 어노테이션을 지원하는데 그 경우에는 그 내용으로 테스트 결과가 찍힌다.

# 12. Event API 구현 : Bad Request 응답 본문 만들기

이제 폼에서 내려주는 에러 데이터에 objectName, field, defaultMessage, code, rejectedValue 들을 내려주고 싶다. 우선 테스트에 .andExpect(jsonPath(“\$[0].objectName”).exists() 뭐 이런식으로 추가해두자. 테스트는 물론 에러가 뜬다.

그리고 EventController 에 브레이크 포인트를 찍고 디버깅을 해보자. 그러면 errors 에 basePrice 나 요런 것들이 들어가 있다. Codes 들도 기본적으로 들어가 있다. 이 에러를 넘겨주면 되는데, body 에 담아서 전달하면 되지 않을까? 안 된다.

Event 는 되고, Errors는 왜 안되는가? -> Java Bean 스펙을 따르는 Event 는 BeanSerializer 를 탈 수 있는데, Errors는 불가능하다. Produces 에 HAL_JSON 으로 명시했기 때문에 변환을 시도 했으나 Errors는 변경할 수 없기 때문에 에러가 난 것이었다.

그래서 ErrorsSerializer 를 만들어서 해결하자. Public class ErrorsSerializer extends JsonSerializer<Errors> { @Override public void serialize(Errors errors, JsonGenerator gen, SerializerProvider serializeProvider) {
뭐 이런 메소드 오버라이드 해야 하는데, 우리는 에러를 여러 개 보내줄 거니까 gen.writeStartArray(); 와 gen.writeEndArray(); 사이에 작성하자.

에러는 크게 필드 에러와 글로벌 에러가 있다. 필드는 한 필드가 문제, 글로벌은 종합적인 문제
필드 에러(한 값) : rejectValue(field, errorCode, defaultMessage)
글로벌 에러(여러 값) : reject(errorCode, defaultMessage)

이제 errors의 글로벌과 필드 에러 둘다를 담아줘야 한다.
errors.getFieldErrors().forEach(e -> {
try {
gen.writeStartObject();
gen.writeStringField("field", e.getField());
gen.writeStringField("objectName", e.getObjectName());
gen.writeStringField("code", e.getCode());
gen.writeStringField("defaultMessage", e.getDefaultMessage());
Object rejectedValue = e.getRejectedValue();
if (rejectedValue != null ) {
gen.writeStringField("rejectedValue", rejectedValue.toString());
}
gen.writeEndObject();
} catch (IOException e1) {
e1.printStackTrace();
}
});

뭐 이런 식으로 에러들에 대해 Json serialize 시 어떤 필드를 넣어줄건지 결정해주는 것이다. 이렇게 작성을 하고, ErrorsSerializer 를 등록해줘야 하는데, 그 컴포넌트를 스프링에서 제공한다. @JsonComponent 만 적으면 Errors 를 Serialize 할 시 가져와서 쓴다. 참 쉽다.

# 13. Event API 구현 : 비즈니스 로직 적용

Event 의 basePrice 와 maxPrice 에 따라 free 의 값 변경
Location 존재 여부에 따라 offline 값 변경 등의 로직을 추가해보자.

이런 비지니스 로직은 도메인 객체 테스트에 추가하는 게 좋다. (컨트롤러 단 보다 훨씬 명확해지니까) Event 가 update 라는 메소드를 실행하면 basePrice 와 maxPrice 를 비교해서 둘 다 0 일 때 넣어준다는 로직을 넣는다. 그리고 컨트롤러에도 mapping 다음 update 를 호출해줘야만 업데이트가 된다. 이런 코드는 서비스로 위임하는 게 좋다. 지금은 간단해서 그냥 넣는다.

Public void update() { this.free = this.basePrice == 0 && this.maxPrice == 0; this.offline = this.location.isBlank(); (java 11) this.location != null && !this.location.trim().isEmpty(); 뭐 이렇게 한다.

# 14. Event API 구현 : 매개변수를 이용한 테스트

테스트 코드 리팩토링 하자. 계속 반복된다!
테스트에서 매개변수만 바꿀 수 있으면 될 것 같은데.. JUnitParams 라는 걸 쓰자.
pl.pragmatists::JUnitParams 를 디펜던시에 추가한다.

EventTest 에 @Runwith(JUnitParamsRunner.class) 를 추가한다.
void testFree 라는 테스트에 적용해본다고 하면, 변하는 파라미터가 basePrice maxPrice 으로 isFree를 체크한다. 따라서 메소드에 파라미터를 void testFree(int basePrice, int maxPrice, boolean isFree) 처럼 받는 걸로 변경한다. 이제 예측하는 값을 어노테이션에 추가한다.
@Parameters({
"0, 0, true",
"100, 0, false",
"0, 100, false",
})

@Parameters({ “0, 0, true”, “100, 0, false”, “0, 100, false”}) 를 넣으면 되는데, 타입 safe 하지 않다. Private Object[] parametersForTestFree() { 를 따서 Object[] 를 반환하게 해서 typeSafe 하게 할 수도 있다.
private Object[] paramsForTestFree() {
return new Object[][] {
new Object[] {0, 0, true},
new Object[] {100, 0, false},
new Object[] {0, 100, false},
new Object[] {100, 200, false},
};
}
@Test
@Parameters(method = "paramsForTestFree")

또한 네이밍을 그냥 parametersForTestFree 라고 하면 알아서 가져간다.

# 15. HATEOAS

REST 어플리케이션 아키텍쳐 중 하나의 요소.
Hypermedia As The Engine Of Application State
하이퍼미디어를 사용해서 애플리케이션 서버간의 정보를 동적으로 주고 받는 방법.

예를 들어 GET /accounts/12345 의 응답에 어카운트 넘버, 잔액 과 다른 액션을 할 수 있는 link들을 내려준다. 다른 상호작용으로의 전이가 가능하며, 관계만 보고 소통을 하기 때문에 uri 가 변경되어도 상관없다. 또한 애플리케이션의 상태에 따라 링크를 내려주는게 달라진다.

스프링 HATEOAS

- 링크
  - HREF : uri
  - REL : 해당 링크와의 릴레이션 등을 보여준다.
    - self : 링크 자신
    - Profile : 설명 문서
    - update-event
    - Query-events
    - ...
- 링크 만드는 기능
  - 문자열 가지고 만들기
  - 컨트롤러와 메소드로 만들기
- 리소스 만드는 기능
  - 리소스 : 데이터 + 링크
  - 리소스는 데이터랑 링크 둘 다 있어야 리소스다.
- 링크 찾아주는 기능
  - Traverson
  - LinkDiscoverers

링크 만드는 건 로케이션에 넣을 헤더를 만들기 위해 이미 썼었다. linkTo(EventController.class).slash(newEvent.getId()).toUri(); 뭐 이런식으로 씀.
만든 링크에 릴레이션, href 등을 설정할 수 있다. 컨트롤러에 매핑된 정보를 가지고 올 수도 있고, 해당 컨트롤러가 가지고 있는 맵핑 정보를 이용해서 링크를 만들수도 있다.

자, 이제 HATEOAS 내리러 가자

# 16. HATEOAS 적용

스프링 부트에 HATEOAS 있으니까 그냥 쓸 수 있음.

우선 테스트에 jsonPath(“\_link.self”).exists() 를 만들어놓자.
그리고 리소스를 내려주기 위한 클래스를 만들자. ResourceSupport 라는 클래스를 상속받아서 구현할 수도 있다.
Class EventResource extends ResourceSupport {
Private Event event; public EventResouce(Event event) { this.event = event };
}

그리고 컨트롤러에 이렇게 EventResouce를 써서 링크를 추가해주자.
ControllerLinkBuilder selfLinkBuilder = linkTo(EventController.class).slash(newEvent.getId());
URI createdUri = selfLinkBuilder.toUri();
EventResource eventResource = new EventResource(newEvent);
eventResource.add(linkTo(EventController.class).withRel("query-events"));
eventResource.add(selfLinkBuilder.withSelfRel());
eventResource.add(selfLinkBuilder.withRel("update-event"));
그런데 문제는 응답이 { “event”: {}, “\_links”: {} } 이러한 형태로 내려간다.

왜냐면 EventResource 를 응답으로 내려줄 때 오브젝트 매퍼가 serialize 를 해준다. 그 때 Bean Serializer 를 쓰는데, Composite Value 이므로 Event 필드의 이름 아래에 넣어준 것이다. 이렇게 감싸고 싶지 않다면, @JsonUnwrapped 라는 어노테이션을 붙이면 된다. Private Event 위에 붙여주자. 그러면 하위에 내려주게 된다.

이걸 더 간단하게 하고 싶으면, ResourceSupport 아래에 Resource<T>라는 것이 있는데, 걔는 기본으로 T는 @JsonUnwrapped 시켜준다. 그래서 Class EventResource extends Resource<Event> 로 변경해주면 멤버 변수와 컨스트럭터를 넣지 않고도 동작해서 간결해졌다. 이 클래스는 빈으로 등록하지 않고 계속 컨버팅할 때 쓴다.

셀프 링크 같은 건 계속 쓰니까 EventResource 에 추가하는 게 나을 것 같다.
add(linkTo(EventController.class).slash(event.getId()).withSelfRel()) 뭐 이런식으로~

# 17. Rest Docs

스프링 REST 도큐먼트 자동으로 생성해주는 툴이다. 아스키 닥터라는 툴을 쓰는데 plain text를 html 로 만들어서 내려준다. MockMvc 도 적용가능하고, 스프링 5에서 지원하는 WebTestClient 에서도 적용 가능하다.

REST Docs를 설정하려면, 테스트위에 @AutoConfigureRestDocs 만 붙이면 된다. 갓부트..

그리고 사용하는 법은 .andDo(document(식별자, snippets…) 를 하면 되는데, 그러면 target/generated-snippets/식별자 에 생성이 된다. 스웨거를 써서 어노테이션으로 가져와도 되는데, 컨트롤러 코드가 바뀔 때 테스트 코드가 바뀌어야 하는데 그 때 문서가 자동으로 바뀌어진다. 또한 테스트가 추가되었을 때 문서를 바꾸도록 강제할 수 있다. 그런 문제 발생 방지를 할 수 있어서 Rest Docs 가 더 좋은 것 같다.

Rest Docs 코딩

- andDo(document(“doc-name”, snippets)
- snippets
  - Links()
  - requestParameters() + parameterWithName()
  - pathParameters() + parametersWithName()
  - requestParts() + partWithname()
  - requestPartBody()
  - requestPartFields()
  - requestHeaders() + headerWithName()
  - requestFields() + fieldWithPath()
  - responseHeaders() + headerWithName()
  - responseFields() + fieldWithPath()
- Relaxed
- 등등이 있다고 한다.. 나중에 알아보자

# 18. Rest Docs 적용

맨 처음에 의존성은 넣어놨으니까, 테스트 위에 @AutoConfigureRestDocs를 추가.

.andDo(document(“create-event”)) 를 붙여서 테스트를 돌려보면, 아무 스니펫도 넣지 않았는데 문서가 생성되어있다! 기본적인 요청들에 대해 예제가 저장된다. 파일을 보면 조금 길게 보기 불편하게 되어 있다. 이걸 커스터마이징 해주자.

RestDocsMockMvcConfigurationCustomizer 를 구현한 빈을 등록하자.
테스트에 common 이라는 패키지를 만들고 public class RestDocsConfiguration 로 만든 다음에 configure 에서 prettyPrint() 넣어주면 되는데, 이런 거 못 외운다. 나중에 찾아보면서 해라. 그리고 테스트에 가서 임포트 시켜주자. @Import(RestDocsConfiguration.class)

저 중간에 넣어주는 걸 preprocesss 라고 하는데, 필요하면 더 찾아서 알아서 넣자.
이제 요청 본문이랑 응답 본문은 문서화가 되었는데, 링크나 응답 헤더 그리고 프로파일 링크에 대한 문서를 추가해줘야 한다.

- 요청 본문 문서화
- 응답 본문 문서화

# 19. Rest Docs : 링크, (Req, Res) 필드와 헤더 문서화

이벤트를 생성하는 테스트를 가지고 이벤트 생성 시 필요한 요청의 필드 정보와 응답의 헤더, 필드, 링크들이 무슨 의미를 가지고 있는지 알려주자.

- 링크 문서화
  - self
  - query-events
  - update-event
  - Profile 링크 추가 // 다음에하기~
- 요청 헤더 문서화
- 요청 필드 문서화
- 응답 헤더 문서화
- 응답 필드 문서화

각각의 스니펫들을 추가하면 된다. .andDo(document(“create-event”, 뒤에 붙이자.
links( linkWithRel(“self”).description(“link to self”), linkWithRel(“query-events”).description(“link to query events”), linkWithRel(“update-event”).description(“link to update an exisitng event”),

테스트 돌려보면 links.adoc 이 생겼고 확인할 수 있다.
그 다음으로는 requestHeaders(headerWithName(HttpHeaders.ACCEPT).description(“accept header”) ) ) 이런 느낌으로 넣어주면 된다.
마찬가지로 requestFields(fieldsWithPath(“name”).description(“”)) 으로 만들어주면 된다.

이렇게 하면 되는데, 모든 값들에 대해서 문서화하고 싶지 않다! 나는 이미 위에 links 에 대한 설명을 적어놨다! 그러면 relaxed 라는 prefix 를 앞에 붙이면 된다. 문서 일부분만 테스트하게 된다.

# 20. Rest Docs : 문서 빌드

빌드 플러그인 넣어라.
https://docs.spring.io/spring-restdocs/docs/2.0.3.RELEASE/reference/html5/#getting-started-build-configuration

그리고, 템플릿 파일을 복사해 asciidoc/index.adoc 에 넣자.
그리고 mvn package 를 실행하자. 패키징할 때 테스트 -> 컴파일 을 실행하므로, 테스트를 돌면서 스니펫들이 생기고 컴파일 시 플러그인 때문에 문서가 생성되어 스프링 부트의 스테틱 디렉토리에 들어가진다.

target/generated-docs/index.html 이 생성되었음을 볼 수 있다. 이 문서를 profile 에 넣어주면 self-descriptive 를 만족시킬 수 있다. 이 파일이 target/classes/static/docs/index.html 에도 들어가 있다. 웹 서버를 띄워서 docs/index.html 을 조회해보면 문서가 떠있음을 볼 수 있다. 해당 설정들이 우리가 넣어준 plugin 에서 outputDirectory, resources directory 들에 설정이 되어 있다. 정적 리소스를 만들고, 옮기고 등 명령어의 순서가 중요하다.

얘네는 빌드되어있을 때 만들어진다. Main 의 resources 로 생각하지마라. 빌드 폴더에서 일어나는 일이다.

이제 profile 링크를 내려주자.
eventResource.add(new Link("/docs/index.html#resources-events-create").withRel("profile"));
이제 HATEOAS 를 만족한다.

# 21. PostgreSQL 적용 (실 DB, 테스트 DB 분리)

- PostgreSQL 의존성 추가. org.postgresql:postgresql
- 도커로 postgresql 컨테이너 실행
  - docker run --name ndb -p 5432:5432 -e POSTGRES_PASSWORD=pass -d postgres
- 도커 컨테이너에 들어가기
  - Docker exec -i -t nab bash
  - Su - postgres
  - psql -d postgres -U postgres
  - \l ( DB 조회)
  - \dt (테이블 조회)
- 프로퍼티에 해당 값 설정
  - spring.datasource.username=postgres
  - spring.datasource.password=pass
  - spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
  - spring.datasource.driver-class-name=org.postgresql.Driver
- 하이버네이트 설정
  - spring.jpa.hibernate.ddl-auto=create-drop
  - spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
  - spring.jpa.properties.hibernate.format_sql=true
    - Sql 을 포매팅 해줌
  - logging.level.org.hibernate.SQL=DEBUG
    - 쿼리가 보임
  - logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
    - 쿼리의 실제 값을 보여줌.
- 테스트 DB 설정
  - 테스트할 때도 같은 DB 쓰게 되어 있다. @SpringApplication 의 설정을 다 따오기 때문에 application.properties 를 다 가져오기 때문이다.
  - 그래서 따로 설정을 해주자!
  - Test/resources 라는 폴더를 만들고, 커맨드 + ; 을 눌러서 모듈에서 해당 폴더를 테스트 리소스 폴더라고 지정해주자.
  - 문제는, 파일 이름이 똑같으면 완전히 파일을 오버라이드 시키기 때문에 추가 설정한 것들이 아예 날라간다. 중복으로 계속 설정해야 한다. 나는 그냥 datasource 설정만 오버라이드 하고 싶다!
  - 그러면 application-test.properties 로 파일 이름을 변경하고 매 테스트마다 선언을 해줘야 한다.
  - 테스트 위에 @ActiveProfiles(“test”) 를 테스트 위에 붙여준다. 테스트 프로파일이니까 저 프로퍼티를 가져가서 사용해준다.
  - 중복을 줄이면서 공용으로 사용되는 것들을 많이 해놨따.

# 22. 인덱스 만들기

모든 홈페이지에서는 클릭으로 이동을 할 수 있다.
즉, 진입점을 통해 다른 리소스에 대해 접근할 수 있다.

인덱스 만들기

- 다른 리소스에 대한 링크 제공
- 문서화

우선 테스트로 만들자. indexContorllerTests 를 만든 다음, /api/ 로 요청을 날리면 \_links.events 가 jsonPath 로 오길 원한다. 그리고 컨트롤러를 만들어서ResourceSupport 를 만들어서 linkTo(EventController.class).withRel(“events”) 로 내려주자.

이런 인덱스 페이지를 에러 페이지에서 활용할 수 있다. 404 에러를 줬을 때 다른 곳으로 전이할 수 있도록 하자. 따라서, 에러를 리소스로 만들어서 link 정보를 내려줘야 한다. ErrorsResource extends Resource<Errors> { 를 만들자. 그리고 constructor 에 add(linkTo(methodOn(IndexController.class).index()).withRel(“index”)); 를 추가하자.

그런데 이 경우, Errors 가 unwrapped 되어 있지 않게 내려온다. 이유는 Resource<Type> 이 @Unwrapped 어노테이션이 있지만 json array 에 대해서는 적용이 되지 않는다고 한다. 따라서 content[0]으로 변경하자.

# 23. Event 목록 조회 API

언제나 그랬듯, 테스트 먼저 만들어보자.
@Test @TestDescription(“30개 이벤트를 10개씩 두번쨰 페이지 조회하기”) Public void queryEvents 에서, 여러 이벤트를 만든 다음 검색이 잘 되는지 요청해봐야 한다.

우선 테스트에 데이터를 만들자. IntStream.range(0,30).forEach(this::generateEvent); 그리고 요청을 다음과 같이 보내자.
this.mockMvc.perform(get(“/api/events”)).param(“page”, “1”).param(“size”, 10).andExpect(status().isOk());

페이징, 정렬은 스프링 데이터 JPA 가 제공하는 Pageable 을 쓰면 된다!!
요걸 쓰면, 페이지와 관련된 많은 파라미터들을 받아올 수 있다. Pagesize, sort 등등..

저걸 써서 컨트롤러에 메소드를 만들어보자. Public ResponseEntity queryEvents(Pageable pageable) { return ResponseEntity.ok(this.eventRepository.findAll(pageable));

그런데 우리가 이전 페이지나 다음 페이지에 대한 링크 정보를 안 내려줬다. 이 때 편한 게 페이지를 리소스로 바꿔서 내려주는 거 말고, PageResourceAssembler<Event> 라는 게 있다.
public ResponseEntity queryEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler) {
Page<Event> page = this.eventRepository.findAll(pageable);
PagedResources<Resource<Event>> pagedResources = assembler.toResource(page);
return ResponseEntity.ok(pagedResources);
이게 있으면 페이지 요청을 리소스에 담아서 내려준다. 그러면 링크를 자동으로 찍어내주기 때문에 개꿀이다.

이제 이벤트목록과 Sort와 페이징, Links 까지 내려주게 되었다.

그런데 아직 완벽한 HATEOAS 가 아니다. 각각의 이벤트에 대한 링크가 없다. 24 번 아이템의 이벤트로 가려면?

그래서, 각 이벤트를 또 리소스로 만들어주면 된다.
PagedResources<Resource<Event>> pagedResources = assembler.toResource(page, e -> new EventResource(e));

이렇게 뒤에 리소스로 변경해주면 이제 각 이벤트에 각자의 \_links.self 가 박혀온다. 따라서 "\_embedded.eventList[0].\_links”는 존재한다는 테스트를 추가하자.

그리고 이제, 페이지 검색한 것의 프로파일과 자기 자신의 링크를 내려줘야 한다. 저 PageResourceAssembler 가 주는 건 마지막, 처음, 다음, 끝 페이지 링크만 준다.

# 24. Event 조회 API

이제 한 이벤트만 조회하는 API를 만들어보자. 이벤트를 만들고, 해당하는 이벤트의 아이디를 가지고 /api/events/{id} 에 요청을 날리자. 원래 유저 정보 받아서 update 링크를 주는지 안 주는지 결정해야 하지만, 이건 나중으로 미루자.

그럼 이제 getEvent 를 구현해보자.
@GetMapping("/{id}")
public ResponseEntity getEvent(@PathVariable Integer id) {
Optional<Event> optionalEvent = this.eventRepository.findById(id);
if (!optionalEvent.isPresent()) {
return ResponseEntity.notFound().build();
}

Event event = optionalEvent.get();
EventResource eventResource = new EventResource(event);
eventResource.add(new Link("/docs/index.html#resources-events-get").withRel("profile"));
return ResponseEntity.ok(eventResource);
}

이런 느낌으로 이벤트를 찾아서 없으면 notFound 내려주면 된다. 참고로, 도큐먼트 뒤에 resources-get-event 같은 정보는 index.adoc 에 정의해놨으니 거기 참고하면 된다. 이제 유저, 인증을 넣어서 좀 더 확장시켜보자.

# 25. Events 수정 API

지금까지 만들었던 수정, 목록, 하나 조회 API를 모든 내용을 넣는 예제이다.

수정하려는 이벤트가 없으면 404
입력 데이터 (데이터 바인딩) 이 이상하면 400
도메인 로직으로 데이터 검증이 실패하면 400
권한이 충분하지 않으면 403
정상적으로 수정한 경우에 200 OK + 수정한 이벤트 데이터

우선, 정상적으로 만드는 테스트를 해보자.
EventDto 를 써서 해야 된다. 왜냐면 애초에 리퀘스트를 날릴 때 EventDto 를 파라미터로 줄 것이기 때문에 이거 하나 만들어서 modelMapper 로 매핑하고 이것저것 하자.

modelMapper(source, destination) 이다. 즉, source 를 destination 에 부어준다.
Event existingEvent = optionalEvent.get();
this.modelMapper.map(eventDto, existingEvent);
Event savedEvent = this.modelMapper.map(existingEvent, Event.class);
savedEvent.update();
Event newEvent = this.eventRepository.save(savedEvent);

이렇게 하면, 존재하는 이벤트를 가지고 와서, eventDto 를 부어준다. 즉 업데이트 시킨 다음에 그걸 Repository 에 저장해준다.

# 26. 테스트 코드 리팩토링

여러 컨트롤러 간의 중복 코드 제거하기

EventControllerTests 와 IndexControllerTests 는 둘다 많은 어노테이션을 가지고 있고, 똑같이 MockMvc 를 주입받아서 사용한다.
이런 중복을 제거하기 위해서는

- 클래스 상속을 사용하는 방법
  - BaseControllerTests를 만들자.
  - 그리고 어노테이션을 다 가져다 붙이고, @Ignore 를 붙여주자. 테스트를 가지고 있지 않기 때문에 붙여줘야 한다.
  - protected MockMvc 를 주입받고, 추가적으로 objectMapper 랑 modelMapper 도 주입받자. (많이 쓰니까)
  - 다른 컨트롤러에 상속시키자.

그리고 모든 테스트가 작동하는지 확인해보자. Mvn test 를 돌리거나, 테스트 페키지를 선택해서 run > All tests를 돌리자.

- @Ignore 때문에 무시되는 게 신경쓰인다면, abstract 클래스로 만들어서 상속 시켜버리자~

# 27. Account 도메인 추가

인증된 사용자만 이벤트를 만들고, 수정하도록 하자.
GrantType 이라고, 인증의 방법에는 여러가지가 있는데 우리는 패스워드 써서 할 거다.

유저라는 객체를 만들고 JPA 맵핑을 하려면 예약어라서 안된다. 물론 이름을 따로 쓸 수도 있지만, 그냥 Account 로 만들자.
@ElementCollection : 여러개의 Enum 값을 가질 수 있으므로 넣어줘야 한다. 그리고 역할은 항상 가져와야 하니까 fetch = FetchType.EAGER 로 설정하자.

그리고 Event 에서 owner 를 조회할 수 있도록 단방향으로 참조해보자.
@ManyToOne
private Account manager;

이렇게 Account 도메인을 추가했고, 스프링 시큐리티를 구현해보자!

# 28. 스프링 시큐리티

스프링 시큐리티는 크게 두 가지 기능이 있다. 두 가지 다 SecurityInterceptor 라는 인터페이스로 사용한다. 인증(유저 확인) / 인가(권한 확인)

- 웹 시큐리티 : 웹 요청에 보안 처리
- 메소드 시큐리티 : 메소드 호출 시 인증 또는 권한 확인

웹 요청 같은 경우는 Servlet filter 랑 연관이 된다. 웹 플럭스 말고 서블릿에 가정해서 이야기하겠다. 메소드 시큐리티는 인터셉터의 역할이다.

서블릿 필터에서 SecurityInterceptor 로 요청을 넘겨서 필터를 적용해야 하는지 아닌지 확인한다. 그리고 만약 필요하다면 인증정보를 SecurityContextHolder 에서 확인한다. 요거의 구현체는 보통 ThreadLocal인데, 쓰레드 내에서 자원을 공유한다. 따라서 파라미터를 쓰레드 로컬에서 가져올 수도 있다. 유저가 로그인하는 동안 공유하는 자원을 가져가서 인증에 쓸 수 있다~~~ 이말이다.

이러한 ThreadLocal을 구현체로 쓰는 SecurityContextHolder 에서 인증 정보를 꺼내서 없으면 유저가 없다! 그러면 AuthenticationManager 로 로그인을 한다. 이 때 사용하는 구현체는 UserDetailsService 와 PasswordEncoder 이다.

AuthenticationManager로 여러가지 인증을 할 수 있는데, BASIC 인증 같은 경우는 인증 요청 헤더에 Authentication basic + username + password 를 인코딩 시켜서 보내줘야 한다. 그 정보를 가지고 PasswordEncoder 를 통해 유저 확인을 한다. 그리고 로그인한 정보를 SecurityContextHolder 에 저장해둔다.

인증이 되었다면, 권한이 적절한지 AccessDecisionManager를 통해 확인한다. 유저의 역할이 리소스에 접근하기 충분한가를 확인하자! 유저에게 Role 을 담아두는데, 저번에 만들었던 AccoutRole 을 체크하자.

이제 요걸 하려면, security 를 의존성에 추가하자.
Org.springframework.security.oauth.boot:spring-security-oauth2-autoconfigure

그리고 UserDetailsService 를 구현한 AccountService 를 만들어보자.
@Service public class AccountService implements UserDetailsService 를 만들고, 테스트를 만들어보자. 마찬가지로 test properties 를 사용할거니까 @ActiveProfiles(“test”) 를 씌운다.

테스트에서 Account 를 만든다음에, ADMIN과 USER 롤을 준다. (Hierarchy 를 통해 ADMIN만 줄수도 있지만 이건 나중에 다루겠다) 그리고 (UserDetailsService)accountService 를 가지고 와서 loadUserByUsername(“keesun”)을 해보자.

요렇게 하기 전에, AccountRepository extends JpaRepository 를 통해 유저를 저장하고, AccountService 에 accountRepository 를 가져와서

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
Account account = accountRepository.findByEmail(username)
.orElseThrow(() -> new UsernameNotFoundException(username));

다음과 같이 처리해준다. 물론 Optional<Account> findByEmail 형태이므로 Optional<T>::orElseThrow가 가능하다. 그리고 나서 이 Account를 UserDetails 로 변환해서 넘겨줘야지 SpringSecurity 가 알아 듣는다.

그러면 UserDetails 위의 클래스인 User에 넣어보자.

        return new User(account.getEmail(), account.getPassword(), authorities(account.getRoles()));

}

private Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
return roles.stream()
.map(r -> new SimpleGrantedAuthority("ROLE\_" + r.name()))
.collect(Collectors.toSet());
}

다음과 같이 매핑을 시켜줬다. 스프링 시큐리티의 인터페이스로 변환 시켜준 것으로 이해하면 된다.

# 29. 예외 테스트

이제 추가적으로 예외 케이스에 대한 테스트도 만들자.
@Test(expected = UsernameNotFoundException.class)
를 붙여서 없는 유저를 찾아보자. 그런데 이렇게만 하면 예외 클래스 타입 외에는 아무런 검증을 하지 못한다.

이 방법 말고 추가적으로 테스트하기 위해 try catch 문을 써보자. 예외가 안 나고 성공하면 fail(“supposed to be failed”); 라는 걸 붙여줘야 하지만 에러 메세지까지 확인할 수 있으니 조금 더 디테일하다.
@Test
public void findByUsernameFail_notFound() {
String username = "random@email.com";
try {
this.accountService.loadUserByUsername(username);
fail("supposed to be failed");
} catch(UsernameNotFoundException e) {
assertThat(e.getMessage()).containsSequence(username);
}
}

이거 말고 다른 방법은 JUnit 의 ExpectedException 이란 Rule이 있다. 이걸 통해서 예외를 미리! 선언해주고 정의해준다.
@Test
public void findByUsernameFail_notFound() {
String username = "random@email.com";
expectedException.expect(UsernameNotFoundException.class);
expectedException.expectMessage(Matchers.containsString(username));

this.accountService.loadUserByUsername(username);
}

꼭, 미리 선언해두고 예외가 던져져야 정상적으로 작동한다.

# 30. 스프링 시큐리티 기본 설정

의존 설정을 추가하는 순간부터 모든 요청은 인증이 필요하다.
스프링 시큐리티의 자동 설정에 의해 모든 요청이 인증이 필요하도록 되어있으니까 모든 테스트가 깨질 것이다.

스프링 시큐리티를 설정하기 전에 우리의 필요 조건을 정리해보자.

- 시큐리티 필터 x : /docs/index.html
- 로그인 없이 접근 가능 : GET /api/events & GET /api/events/{id}
- 로그인 해야 접근 가능 : 나머지 다 & POST /api/events & PUT /api/events/{id}

이렇게 하기 위해서는 스프링 시큐리티 OAuth 2.0 을 사용할 건데, Autrhorization Server 와 Resource Server에서 공통적으로 사용할 스프링 시큐리티 설정을 해야 한다.

/configs/SecurityConfig 클래스를 만들자. @Configuration @EnableWebSecurity public class Security extends WebSecurityConfigurerAdapter {} 를 만들자. 그러면 스프링 부트가 제공하는 스프링 시큐리티 설정이 오버라이드 된다. 그리고 패스워드 인코더와 모델 매퍼를 등록하기 위해서 @Configuration public class AppConfig 클래스도 하나 만들자. 그리고 @Bean PasswordEncoder 를 등록해주자.

그리고 SecurityConfig에서 AuthenticationManager 인 UserDetailsService를 가져오자. 그리고 PasswordEncoder 도 가져오자.

그 다음, OAuth 토큰을 저장하는 곳을 빈으로 등록하자. @Bean public TokenStore tokenStore() { return new InMemoryTokenStore(); }

그리고 AuthenticationManager 를 빈으로 노출시켜줘야 하기 때문에, 다른 Auth 서버 혹은 Resource 서버가 참조하기 위해 generator(커맨드 + N) 누르고 override methods 중 authenticationManagerBean 을 찾아서 정의하자.

그리고 이 AuthenticationManagerBuilder 를 어떻게 정의해야 할지 정해야 하기 때문에 configure 메소드도 오버라이드 하고 정의해보자.

auth.userDetailsService(accountService).passwordEncoder(passwordEncoder);

userDetailsService 와 passwordEncoder 둘 다 내가 정의한 걸로 쓰게 만들자.

그리고 이제 필터를 적용할지 말지 설정해보자. configure(WebSecurity web) 을 오버라이드 메쏘드 한 다음, 기본 리소스 위치에는 적용되지 않고, 기본 독도 설정되지 않게 한다.
@Override
public void configure(WebSecurity web) throws Exception {
web.ignoring().mvcMatchers("/docs/index.html");
web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
}

요렇게 하면 docs/index 랑, 기본 리소스에는 접근이 가능하다.

그리고 http 에서 걸러보자. 요청의 필요로 하는 인증이 anonymous 다. 그러면 아무나 접근할 수 있는 건데, authorize 할 때 docs와 스태틱 리소스들은 는 아무나 접근 가능하다로 설정하려면, 아래와 같이 해주면 된다.

@Override
protected void configure(HttpSecurity http) throws Exception {
http.authorizeRequests().mvcMatchers("/docs/index.html").anonymous()
.requestMatchers(PathRequest.toStaticResources().atCommonLocations()).anonymous()
;
}

어차피 걸러줄 거면, http 가 아닌 웹에서 걸러주는 게 편하니까 http configure 말고 web configure 를 하자.

스프링 시큐리티 설정 요약

- @EnableWebSecurity
- @EnableGlobalMethodSecurity
- extends WebSecurityConfigurerAdapter
- PasswordEncoder: PasswordEncoderFactories.createDelegatingPassworkEncoder()
- TokenStore: InMemoryTokenStore
- AuthenticationManagerBean
- configure(AuthenticationManagerBuidler auth)
  - userDetailsService
  - passwordEncoder
- configure(HttpSecurity http)
  - /docs/\*\*: permitAll
- ● configure(WebSecurty web)
  - ignore
    - /docs/\*\*
    - /favicon.ico
- PathRequest.toStaticResources() 사용하기 (스태틱 리소스 표현)

# 31. 스프링 시큐리티 폼 인증 설정

@Override
protected void configure(HttpSecurity http) throws Exception {
http.anonymous()
.and()
.formLogin()
.and()
.authorizeRequests()
.mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
.anyRequest().authenticated();
}
configs/SecurityConfig.java
익명 사용자를 허용할 것이고, 폼 인증도 사용할 것이고(로그인 페이지도 설정할 수 있지만 스프링은 이제 기본 로그인 페이지도 있으니 생략), GET 요청으로 /api/** 를 익명으로 접근하는 것을 허용하고, 다른 요청들은 다 인증을 받아야만 한다.
는 설정이다.

그런데 우리가 위에서 user 찾을 때 패스워드 인코더를 쓰는데, 리포지토리에 유저 저장할 때도 인코더를 써야 하니까 그 쪽 설정을 하러 가자.

AccountService 에 saveAccount 를 만들고, 거기서 this.passwordEncoder.encode 를 통해 감아서 넣어주자. 그리고 테스트에서도 this.passwordEncoder.matches() 를 통해 비밀번호 체크를 할 수 있다. 폼 인증 설정하는 방법이었다.
