
# java-spring-message

우아한형제들 김영한 님의 인프런 강의 '스프링 MVC 2편 - 백엔드 웹 개발 활용 기술'의 3장, "메시지, 국제화" 부분을 따라치면서 정리하는 Repository

---

## 메시지 소스 등록

- 빈 등록
- 스프링 부트 기준으로는 기본등록됨
  - application.properties에서 수정하여 설정을 추가, 변경 가능

## 수동 빈 등록
```java
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new
            ResourceBundleMessageSource();
    messageSource.setBasenames("messages", "errors");
    messageSource.setDefaultEncoding("utf-8");
    return messageSource;
}
```
- MessageSource를 수동 빈 등록
  - setBasenames : 메시지 소스 파일명들을 가변인자로 등록
  - setDefaultEncoding : 문자 인코딩을 지정(보통 utf-8)
- 스프링부트에서는 기본적으로 자동으로 messageSource를 빈을 등록해줌

## 스프링 부트의 메시지 소스 설정
```properties
# 스프링부트 메시지 소스 기본 값
spring.messages.basename=messages
# 콤마를 통해 수동으로 여러가지 메시지 소스를 등록/변경 가능.
spring.messages.basename=messages,config.i18n.messages
```
- application.properties에서 메시지 소스 설정을 수동으로 할 수 있음
- 기본적으로 messages를 메시지 소스로 등록함

---

## MessageSource 클래스
```java
public interface MessageSource {
    
    @Nullable 
    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
	
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;
	
    String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;

}
```
- locale 정보가 없을 경우, basename에서 설정한 기본 이름 메시지 파일을 조회
  - 여기서는 messages를 지정했으므로, messages.properties에서 조회
- getMessage
  - code : 코드
  - args : 인자들
  - defaultMessage : 코드가 없을 경우 반환될 기본 메시지
    - 매칭되는 코드가 없고 defaultMessage가 없을 경우, `NoSuchMessageException`이 던져짐
  - Locale : 로케일
- 국제화 파일 선택
  - Locale에 맞추어 국제화 파일을 선택
    - Locale로 en_US로 지정될 경우, `messages.en_US` -> `messages.en` -> `messages` 순으로 찾음
    - Locale에 맞추어 구체적인 것이 있으면 먼저 찾고 없으면 디폴트까지 순서대로 찾아감

---

## 타임리프 메시지 + 국제화 메시지 적용
```properties
hello=안녕
hello.name=안녕 {0}

label.item=상품
label.item.id=상품 ID
label.item.itemName=상품명
label.item.price=가격
label.item.quantity=수량

page.items=상품 목록
page.item=상품 상세
page.addItem=상품 등록
page.updateItem=상품 수정

button.save=저장
button.cancel=취소
```
```html
  <div class="py-5 text-center">
      <h2 th:text="#{page.addItem}">상품 등록</h2>
  </div>
```
- 타임리프는 `#{...}`를 통해 메시지 소스에 등록된 코드, 메시지를 기반으로 편하게 메시지 처리를 할 수 있게 도와줌
- 요청 Header의 `Accept-Language`에 지정된 언어 설정 값에 따라, 국제화가 적용됨
  - Chrome 기준으로, 설정-고급-언어에서 설정을 변경하면 요청 시 `Accept-Language` 값이 변경된다.

### 타임리프 메시지 - 파라미터 적용
```properties
hello.name=안녕 {0}
```
```html
<p th:text="#{hello.name(${item.itemName})}"></p>
```
- 파라미터를 지정하여, 메시지 설정 가능

### LocaleResolver - 로케일 선택방식 변경

- 스프링은 Locale 선택방식을 변경할 수 있도록 `LocaleResolver` 인터페이스를 지원
- 스프링부트는 기본적으로 `Accept-Language` 헤더를 활용하는 `AcceptHeaderLocaleResolver`를 사용
- 필요에 따라, 쿠키 등을 기반으로 Locale 선택방식을 변경할 수 있도록 LocaleResolver 구현체를 만들어서 로케일 설정을 변경하는 방식을 사용할 수 있다.

---
