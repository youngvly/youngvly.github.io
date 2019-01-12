---
title: "AutoWired vs ConstructorInjection"
date: 2019-01-12 22:37:00 -0400
categories: Spring-boot-study
---
# Field Injection

    Autowired로 필드에 직접 주입하는 방식.

    ```java

    @Component
    public class FieldInjection {
        @Autowired
        private SomeService someService;
    }
         
    ```
**문제점**



# Setter Injection

    Setter에서 주입시키는 방식
    (Spring 3.x에서는 Setter주입을 Constructor주입보다 더 장려한다.)

    ```java

    @Component
    public class SetterInjection {
        private SomeService someService;
        
        public void setSomeService(SomeService s){
            this.someService = s;
        }
    }
    ```
# Constructor Injection

    생성자에서 주입시키는 방식
    (Spring 4.3 에서 단일 생성자의 경우 @AutoWired가 필요없다. -> Spring 4.x에서는 더이상 Setter주입이 추천되지 않는다.)

    ```java

    @Component
    public class ConstructorInjection {
        private final SomeService someService;

        public ConstructorInjection(SomeService s) {
            this.someService = s;
        }
    }
    ```

# Field Injection 보다 Constructor Injection이 권장되는 이유

1. 단일 책임의 원칙
    (SOLID -> S : 모든 클래스는 하나의 책임만을 가지며, 그 책임을 완전히 캡슐화해야한다.)
    생성자의 인자가 많을 경우, 코드량도 많아지고, 의존관계도 많아져 단일 책임의 원칙에 위배된다.
    그래서 Constructor Injection을 사용함으로써 의존관계, 복잡성을 쉽게 알 수 있어 리팩토링의 단초를 제공하게 된다.
2. 테스트 용이성
    DI 컨테이너에서 관리되는 클래스는 특정 DI 컨테이너에 의존하지 않고 POJO이어야 한다
    DI 컨테이너를 사용하지 않고도 인스턴스화 할 수 있고, 단위테스트도 가능하며, 다른 DI 프레임워크로 전환할 수도 있게 된다. 
3. **Immutability (수정불가능변수)**
    생성자 주입에서는 필드를  final로 선언할 수 있다.
    FieldInjection 과 SetterInjection에서는 final로 선언할 수 없기때문에 변경가능한 상태가 된다.
4. 순환 의존성
   생성자 주입에서는 맴버 객체가 순환 의존성을 가질 경우 BeanCurrentlyInCreationException 이 발생해 순환 의존성을 알 수 있게 된다. 
5. 의존성 명시
   생성자 주입을 사용함 으로 써 해당 의존 객체가 필수임을 명시한다. 
   옵션일때는 Setter 주입을 활용할 수 있다. 

+ Field Injection의 문제
  - default 생성자를 호출함으로써, 의존체 오브젝트를 new를 사용하여 할당할 수 있다. 이때 NullPointerException을 발생시킬 수 있다. 
  
## Lombok에서의 Constructor Injection

```java

@RequriedArgsConstructor
@Component
public class ConstructorInjection {
        @NonNull
        private final SomeService someService;
    }
```

- @RequiredArgsConstructor : 초기화되지 않은 final필드를 매개변수로 취하는 생성자를 생성한다.
- @NonNull : 해당 파라미터가 null인경우 NullPointerException을 발생시킨다.  

## Spring 4.3 에서의 ObjectProvider

```java

@Service
public class FooService {
    private final FooRepository repo;
    public FooService(ObjectProvider<FooRepository> repositoryProvider){
        this.repository = repositoryProvider.getIfUnique();
    }
}
```

- ObjectProvider<> : ObjectFactory 인터페이스의 확장형으로 getIfAvailable, getIfUnique함수의 사용이 가능하다. 실제로 존재하는 빈 혹은 유일하게 등록된 하나의 빈을 찾아낼수있다.


*참고*
[1] Spring 4.3 Setter Injection : https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3
[2] Spring에서 Field Injection보다 Construct Injection이 권장되는 이유 : http://www.mimul.com/pebble/default/2018/03/30/1522386129211.html 
[3] Field-dependency-Injection이 위험한 이유 : https://www.vojtechruzicka.com/field-dependency-injection-considered-harmful/ 