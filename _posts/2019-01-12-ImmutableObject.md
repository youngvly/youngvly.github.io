---
title: "Immutable Object (RestTemplate Builder을 Bean으로 생성하기)"
date: 2019-01-12 22:37:00 -0400
categories: Spring-boot-study
---
# RestTemplateBuilder Bean 으로 RestTemplate 생성

1. RestTemplateBuilder를 Bean으로 생성
    RestTemplate의 factory, 등을 설정하기위해 Builder를 Bean으로 설정함.
```java
    @Bean
    public RestTemplateBuilder basicRestTemplateBuilder() {
        RestTemplateBuilder builder = new RestTemplateBuilder();
        builder.errorHandler(new SomeResponseErrorHandler())
                .defaultMessageConverters();
        return builder;
    }
    @Bean
    public RestTemplate basicRestTemplate (RestTemplateBuilder restTemplateBuilder) {
        return restTemplateBuilder.build();
    }
```

+ 문제 : *RestTemplate를 생성하나, 변경된 errorHandler가 적용되지 않음.*
+ 가능성 : **Spring Boot에서 기본으로 제공하는 RestTemplateBuilder Bean이 있음**
RestTemplateBuilder Bean이 없음에도 위의 코드가 실행된다. 
    -> Spring Boot에서 기본으로 제공하는 Builder가 있기때문! (RestTemplateAutoConfiguration.class)
  
2. @Qualifier 적용
```java
    @Bean
    @Qualifier(value = "basicRestTemplateBuilder")
    public RestTemplate basicRestTemplate (RestTemplateBuilder restTemplateBuilder) {
        return restTemplateBuilder.build();
    }
```
+ 문제 : *RestTemplate를 생성하나, 변경된 errorHandler가 적용되지 않음.*
+ 가능성 : **RestTemplateBuilder가 Immutable 한 객체이다.**

# Immutable Object
- 객체지향 프로그래밍에 있어서 불변객체는 생성 후 그 상태를 바꿀 수 없는 객체를 말한다.
- 모든 필드가 final인 클래스
- 객체를 변경하는 setter가 없다.
- 상속 금지. (final class로 정의하면 상속이 금지된다.)
- 가변 객체 참조 필드를 사용자가 얻을 수 없도록 해야한다. (private)
```java 
String s = "a";
s = "b";
```
위의 코드에서 s는 재할당되었다. s가 refrence하고있는 heap영역의 객체가 바뀌었을 뿐, heap영역의 값이 바뀐것은 아니다. -> String은 대표적인 Immutable 클래스 ->  mutable 한 String을 사용하기 위해서는 StringBuffer을 주로 사용한다.


  
**해결방안**
```java 
    @Bean
    public RestTemplateBuilder basicRestTemplateBuilder() {
       return new RestTemplateBuilder()
                .errorHandler(new SomeResponseHandler())
                .defaultMessageConverters();
    }
    @Bean
    public RestTemplate basicRestTemplate (RestTemplateBuilder restTemplateBuilder) {
        return restTemplateBuilder.build();
    }
```
## 장점
    1. 생성자의 방어 복사 및 접근 메소드의 방어 복사가 필요없다.
    2. 병렬 프로그래밍을 작성할 때 , 동기화 없이 객체를 공유가능하다.
        "특별한 이유가 없다면 객체를 불변객체로 설계해야한다."
## 단점
    1. 객체가 가지는 값마다 새로운 객체가 필요하다.
    2. 값이 계속 변경되는곳에 쓰이면 메모리를 많이 소비한다. (Garbage Collection덕분에 치명적인 위험은 아니다.)

## 불변객체 생성
```java 
public class Example{
    private static final Integer[] VALUES = {1,2,3,4,5};
}
```
위의 코드에서 VALUES는 변하지 않지만, VALUES안의 값이 변할 수 있다.

1. static getter 에서 복사본으로 제공한다. 
```java 
public class Example{
    private static final Integer[] VALUES = {1,2,3,4,5};
    public static Integer[] values() {
        return VALUES.clone();
    }
}
```
2. Collections.unmodifiableCollection() 을 사용한다.
   (add,put과 같은 method를 호출시에 UnsupportedOperationException이 발생한다.)
```java 
public class Example{
    private static final Integer[] VALUES_PRIVATE = {1,2,3,4,5};
    public static final Collection<Integer> VALUES = Collections.unmodifiableCollection(Arrays.asList(VALUES_PRIVATE));
}
```

[1] RestTemplate with RestTemplateBuilder : https://stackoverflow.com/questions/45670955/spring-boot-resttemplate-basic-authentication-using-resttemplatebuilder
[2] Effective JAVA : http://dev-ahn.tistory.com/126
[3] java에서 immutable : https://hashcode.co.kr/questions/727/%EC%9E%90%EB%B0%94%EC%97%90%EC%84%9C-immutable%EC%9D%B4-%EB%AD%94%EA%B0%80%EC%9A%94
