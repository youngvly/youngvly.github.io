---
title: "Spring boot Test Mock"
date: 2019-01-12 22:37:00 -0400
categories: SpringBoot TEST
---

# Spring Boot Test

Spring boot 1.4부터 적용되는 Spring Boot Test 어노테이션
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SimpleTestCase() {...}
```
## Test Double
목적에 따라 다른 객체를 사용하는 모든 행위. (Stub, Dummy, Mock, Spy)
- 테스트 대상 코드 격리
- 테스트 속도 개선
- 예측 불가능한 실행 요소 제거
- 특수한 상황 테스트 가능
- 감춰진 정보를 확인 가능. 
  
### Test Stub
+ 의존하는 객체의 기능을 대리하는것, Mock을 생성하는것을 의미한다. 
+ 의존하는것과 독립적으로 개발 및 테스트가 가능하다.

### Mockito
- Mockito 는 JUnit 위에서 동작하는 Mocking과 Verification을 도와주는 프레임워크.
- 단위테스트를 하기 위해 Mock을 만들어주는 프레임워크.

## 1_ @Mock
Mock 객체를 주입한다.
+ Mock이란 껍데기만 있는 가상 객체!
    기존에 사용되던 Bean이 아닌 껍데기만 주입시켜 내부의 구현 부분은 모두 사용자에게 위임.
    Mockito.when().thenReturn(someThing); 으로 동작을 구현시켜주어야한다.
```java
@Mock
private RestTemplate restTemplate;
```
위의 어노테이션을 사용한 결과는 아래와 같다
```java
RestTemplate restTemplate = Mockito.mock(RestTemplate.class);
```
+ 실패 메시지에 필드 이름이 같이 나오기 때문에, mock에서 발생된 문제를 보다 읽기 편하게 해준다. 

## 2_ @Spy
실제 객체를 주입한다.
```java
@Spy
private List<String> groups = new ArrayList();
```

## 3_ @InjectMocks
어떠한 클래스의 instance를 생성하여 @Mock혹은 @Spy로 생성된 Mock들을 해당 인스턴스에 주입한다. 
```java
@Mock
private RestTemplate restTemplate;
@InjectMocks
private SomeController someController;
```
위의 코드는 아래와 같이 동작할 것이다.
```java
SomeController someController = new SomeController(restTemplate);
```

## 4_ @MockBean
- Spring-boot-test에서 제공하는 어노테이션.
- Mock 객체들을 Spring의 ApplicationContext에 넣어준다. (@Mock과의 큰 차이점)
- 둥일한 타입의 Bean이 존재할 경우 MockBean으로 교체해준다.
```java
@Autowired      //ApplicationContext에 올라와있는 Bean자체를 주입
private RestTemplate restTemplate;
@MockBean(name="userService")       //ApplicationContext에 새로 Mock객체를 올리고, 기존에 context에 있던 객체를 덮어쓴다.
public UserService userMockService;
@Test
public void doSomething(){
    Mockito.when(userMockService.getCount()).thenReturn(123L);

    UserService userServiceFromContext = context.getBean(UserService.class);
    long userCount = userServiceFromContext.getCount();

    Assert.assertEquals(123L,userCount);
    Mockito.verify(userMockService).getCount();
}
```

## 5_ @SpyBean
존재하는 Bean을 spy가 감싸는 방식.

- @MockBean은 given에서 선언한 코드 외에는 전부 사용할 수 없다.
- @SpyBean은 given에서 선언한 코드 이외에는 전부 실제 객체의 것을 이용한다.
```java
@SpyBean(name="userService")
public UserService userSpyService;
@Test
public void() {
    String userName = "unknown";
    Mockito.when(userSpyService.findAgeByName(userName)).thenReturn(Optional.empty());

    Assert.assertThat(userSpyService.findAgeByName(userName),is(Optional.Empty())); //위에서 지정한 동작대로 리턴될것
    Assert.assertEquals(userSpyService.getCount(),123L);    //원래 service에서 선언한 함수대로 리턴될것.
} 
```

## 6_ @WebAppConfiguration
```java
    private MockMvc mockMvc;
    @Autowired
    private WebApplicationContext webApplicationContext;

    @Before
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }
```
위의 과정 대신 @AutoConfigureMockMvc 을 사용하면 바로 Mockmvc 사용 가능







1. @Mock
2. @Spy
3. @Captor
4. @InjectMocks

[1] Mockito features in Korean : https://github.com/mockito/mockito/wiki/Mockito-features-in-Korean
[2] https://www.baeldung.com/java-spring-mockito-mock-mockbean