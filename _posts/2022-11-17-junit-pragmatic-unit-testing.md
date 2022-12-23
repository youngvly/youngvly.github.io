---
layout: post
title: "자바와 JUnit을 활용한 실용주의 단위 테스트"
date: 2022-11-17 15:50
categories: Test, JUnit, Java
tags: study, Test
---
## 책 코드 환경
- Java 12
- junit 4.11


# 01 첫번째 Junit test 만들기
- eclipse 에서 junit test guide 내용
- 정상적으로 동작하는지 증명하기 위해 의도적으로 테스트에 실패하라.
-  
# 02 Junit 진짜로 써보기
- `@Before` 어노테이션, 각 테스트가 돌기전에 한번씩 수행된다. 
- 테스트 메서드 명은 직관적으로.
- 각 테스트는 독립적이어아햐며, 테스트 클래스에서는 static 필드를 피해야한다.


# 03 Junit 단언 깊게 파기
- hamcrest = matcher 스펠링 섞음!
- is 메서드는 가독성만 늘려줄뿐.
```java
 assertThat(account.getName(), is(equalTo("name")))
```
- null 여부를 자주 검사하는것은, 설계문제이거나, 지나친 걱정. 가치가 없을 수 있음.
- junit 에 포함된 hamcrest는 일부이고, 직접 import하면 더 많은 Matcher가 존재.
```java
import static org.hamcrest.number.IsCloseTo.*;

assertThat(2.32*3, closeTo(6.96.0.0005)) //  오차 지정
```
- assert 문의 description은 되도록이면 테스트메서드 이름, 의미있는 변수명, 코드개선 등으로 대신하자.

## Exception check
1. Test annotation
    ```java
    @Test(expected=SomeException.class) test(){...}
    ```
2. try/catch/fail
    - checked Exception이 발생하면 실패시 그냥 throw 하자.
    ```java
    try{}
    catch(SomeException e){
        assertThat(e.getMessage(), equalTo("exception Message expected"));  // 예외발생시 성공
    }
    catch(Exception e){
        fail("unknown exception thrown")        //예외 발생시 실패
    }
    ```
3. ExpectedException rule
    ```java
    @Rule
    public ExpectedException thrown = ExpectedException.none()

    @Test
    public test(){
            thrown.expect(SomeException.class);
            thrown.expectMessage("fail Message");

            sut.do();
    }
    ```
4. Fishbowl
    - [fishbowl](https://github.com/stefanbirkner/fishbowl/blob/master/src/test/java8/com/github/stefanbirkner/fishbowl/DefaultIfExceptionDocumentationTest.java)
    ```java
        import static com.github.stefanbirkner.fishbowl.Fishbowl.defaultIfException;
        import static com.github.stefanbirkner.fishbowl.Fishbowl.wrapCheckedException;
    
        public class ExceptionTest {
            @Test
            public void returnsDefaultValueIfExceptionOfSpecifiedTypeIsThrown() {
                long value = defaultIfException(() -> parseLong("NaN"), NumberFormatException.class, 0L);
                assertThat(value, is(0L));
            }

            @Test
            public void wrapsCodeWithReturnValue() {
                URL url = wrapCheckedException(() -> new URL("http://junit.org/"));
            }
        }
   
    ```  

# 04 테스트 조직
### AAA로 테스트 일관성 유지
1. Arrange 준비 : 객체생성, 이 객체와 의사소통하거나 다른 API 호출 등
2. Act 실행 : 단일메서드 호출
3. Assert 단언 : 실행 한 코드가 기대한대로 동작하는지 확인
    - 반환값
    - 객체의 새로운 상태 검증
    - 테스트코드와 다른 객체사이의 의사소통 검증
4. After 사후 (선택적) : 자원 잘 정리되었는지 확인
### 테스트는 개별 메서드를 테스트하는 것이 아니라, 클래스의 종합적인 동작을 테스트해야한다.
### 프로덕션 코드와 테스트 코드
- 프로덕션코드 : 테스트 대상 시스템 (SUT : System Under Test)
- 테스트코드는 프로덕션코드를 의존하지만, 프로덕션 코드는 테스트코드의 존재를 몰라야한다. 
- 테스트코드를 프로덕션 코드의 패키지와 다르게 하면, public 인터페이스만 활용하여 테스트를 작성해야한다.
- sut의 내부 세부사항을 테스트하는것은 테스트 저품질로 이어진다. 테스트 유지보수가 어려워짐.
- sut 내부 데이터 노출 목적이 테스트라면, 테스트와 프로덕션 코드의 과도한 결합을 초래한다.
- 내부 행위를 테스트하려는 것은 설계에 문제가 있다는 것이며, 이때는 private 메서드를 추출하여 별도 클래스의 public 인터페이스로 분리하는 방법이 있다.

### 집중적인 단일 목적 테스트의 가치
- 다수의 케이스를 별도의 테스트 메서드로 분리하고, 메서드 명을 검증하려는 동작을 표현하는 이름을 붙이자.
  - 어느 동작이 실패했는지 빠르게 파악가능
  - Junit은 각 테스트를 별도의 인스턴스로 실행하기때문에 다른 테스트의 영향도를 줄일 수 있다.
  - 모든 케이스가 실행되었음을 보장할 수 있다. 단언이 여러개면 실패한부분에서 중단되기때문.

### 문서로서의 테스트
- 일관성 있는 이름으로 테스트 문서화
  - doingSomeOperationGeneratesSomeResults
  - someResultOccursUnderSomeCondition
  - BDD : (givenSomeContext)whenDoingSomeBehaviorThenSomeResultOpccurs
    - 괄호친 givenSomeContext는 생략가능.
- 테스트를 의미있게 만들기
  - 메서드명, 지역변수명 개선
  - 의미있는 상수 도입
  - 테스트 메서드를 잘게 나누기
  - 테스트 군더더기를 도우미 메서드와 before 메서드로 이동

### @Before와 @After (공통 초기화와 정리) 더 알기
- 초기화가 길어진다면 @Before 를 여러개로 쪼개자. 단 순서보장은 되지 않는다.
- @After는 테스트가 실패해도 실행된다. 
- @BeforeClass, @AfterClass static 메서드이고, 클래스 내 전체 테스트 수행전에 실행된다.
- 순서
  - @BeforeClass
  - @Before
  - @Test
  - @After
  - @Before
  - @Test
  - @After
  - @AfterClass

### 테스트 유지
- 실패하는 테스트가 있다면 테스트를 늘리지말고, 있는것부터 고치자.
- mocking 하여 테스트 속도를 개선하자.
- 최대한 많은 테스트를 돌려보자. (프로젝트단위 > 패키지단위)
- Junit Categories 기능으로 필요한부분만 돌리자.  (= Tag)
- 백그라운드에서 테스트 항상 실행 [Infinitest](https://galid1.tistory.com/615)
  
### 테스트 제외
- 다수의 테스트가 실패한다면, 문제가 있는 테스트만 집중하고(?) @Ignore 

# 05 좋은 테스트의 FIRST 속성
### Fast 빠르다
- 느린 테스트에 대한 의존성을 줄이자.
### Isolated 고립되어있다
- 테스트 순서, 데이터 소스 등 의존성을 없애 테스트를 독립적으로 만들자.
- 테스트가 작은 양의 동작에만 집중하면 독립적으로 유지하기 쉬워진다.
### Repeatable 반복 가능하다
- 실행할때마다 결과가 같아야한다.
- 직접 통제할 수 없는 외부 환경과 격리시키자. ex) timeStamp- 
### Self-validating 스스로 검증 가능하다
- 테스트에 필요한 설정단계는 모두 자동화되어야한다.
- CI/CD 적용, InfiniTest 등 자동실행 솔루션 사용
### Timely 적시에 사용한다
- 옛날코드에 대한 테스트는 시간낭비가 될 수 있다.
- 코드 변경이 있거나, 문제가 발생하는 부분 등 효율적인부분부터 작성하자.
  
# 06 Right-BICEP : 무엇을 테스트할 것인가.
### Right : 결과가 올바른가
### B : Boundary condition (경계조건) 이 맞는가
- 수치적 오버플로우를 일으키는 계산
- null
- 특수문자가 포함된 파일명.
- 경계조건에서는 CORRECT 7장 참고
### I : Inverse relationship (역관계) 을 검사할 수 있는가.
- 논리적인 역관계를 적용하여 행동검사
- 곱셈으로 나눗셈 검증
- DB insert를 DB select로 검사
### C : 다른 수단을 활용하여 Cross-Check 할 수 있는가.
- 동일한 결과를 내는 다른 솔루션 사용 (Newton.squareRoot = Math.sqrt)
### E : Error Condition 을 강제로 일어나게 할 수 있는가.
-  memory, disk full
-  network availability
### P : Performance characteristics (성능 조건) 는 기준에 부합하는가
- 병목지점을 단위테스트를 작성하여 명확하게 파악하고, 개선 후 차이를 파악하자.
- 모든 성능 최적화 시도는 실제 데이터로 해야하며, 추측을 기반으로 해서는안된다.

# 07. 경계조건 : CORRECT 기억법
단위테스트를 만들때 고려해야할 경계조건 기억법
### Conformance : 준수
: 값이 기대한 양식을 준수하고 있는가
### Ordering : 순서
: 값의 집합이 적절하게 정렬되거나 정렬되지 않았나
- `@After` 로 불변성 검사
  - 사용자 정의 Matcher 생성 하여 가독성을 키우는 예
  - 서비스코드 내 불변상태를 검사하는 메서드를 생성하여 테스트코드에서 호출 : 내부 상태 노출을 최소화할 수 있다.
### Range : 범위
: 이성적인 최솟값과 최댓값 안에 있는가
### Reference : 참조
: 코드 자체에서 통제할 수 없는 어떤 외부 참조를 포함하고 있는가
- 반드시 존재해야하는 그 외 다른 조건들
- 범위를 넘어서는 것을 참조하고 있지 않은지
- 외부 의존성은 무엇인지
- 특정 상태에 있는 객체를 의존하고 있는지
- ex) 자동차 변속기, 주행중에 P로 바꾸는 요청을 무시하는가
### Existence : 존재
: 값이 존재하는가 (Null / 0 / collection.exist)
### Cardinality : 기수
: 정확히 충분한 값들이 있는가
- 0-1-n 법칙 : 0개, 1개, n개 있는경우를 테스트하자. 1개만있는경우도 중요하다. n은 비즈니스 요구사항에따라 바뀔수 있다.
### Time : 시간 
: 모든것이 순서대로 일어나는가, 정확한 시간, 정시에?
- 상대적 시간 (시간 순서) : 메서드 실행 순서
- 절대적 시간 (측정된 시간) : 타임아웃
- 동시성 문제들

# 08. 깔끔한 코드로 리팩토링하기
- 테스트는 리팩토링의 안전장치
- 예시코드 참고
  - [ASIS](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_16/src/iloveyouboss/Profile.java)
  - [TOBE](https://github.com/gilbutITbook/006814/blob/master/iloveyouboss_big-6/src/iloveyouboss/Profile.java)
- for문을 3개의 for문으로 쪼개서 가독성을 높이고, 성능을 포기함.
  - 깔끔한 설계는 최적화를 위한 최선의 준비.
  - 성능은 문제가 될때 수정해도 늦지않다

# 09. 더 큰 설계 문제
### SOLID
- SRP 단일책임의 원칙 : 클래스를 변경할때 한가지 이유만 있어야하고, 클래스는 작고 단일 목적을 추구한다.
- OCP 개방 폐쇄 원칙 : 확장에 열리고 변경에 닫혀있어야한다.
- LSP 리스코프 치환 법칙 : 하위 타입은 상위타입을 대체할수있어야한다.
- ISP 인터페이스 분리 원칙 : 의존성을 줄이기 위해 큰 인터페이스를 쪼개자.
- DIP 의존성 역전 원칙 : 고수준 모듈은 저수준 모듈에 의존하면 안되고, 추상클래스에 의존해야한다.
### Command-Query 분리
- 부작용을 주는 명령과, 부작용이 없는 질의는 같이 있으면 안된다.
### 단위테스트의 유지 보수 비용
- 시스템 설계 및 코드 품질이 낮을수록 단위테스트의 유지 보수 비용은 증가한다.
- 클래스를 분할하자.
- private 메서드가 자꾸 늘면, 내부 동작을 새 클래스로 옮기고 public으로 만드는 것이 좋다.
- 테스트에서 코드중복을 피하기위해 중복코드를 도우미 메서드로 분리하면, 변경 영향을 최소화할 수 있고, 이해하기 쉽다.
- 설계를 지속적으로 개선해나가는 자신감을 키우기 위해 단위 테스트의 커버리지를 높이세요




> 단위테스트에서는 전역변수를 쓰지말라고 했던것같은데, 테스트별로 인스턴스가 생성되는거라면 전역변수나 before를 활용하는것이 맞을수도있겠다. before와 전역변수를 적극 활용하는것이 좋은 방법일까?

# 10. mock 객체 사용
- stub : 테스트 용도로 하드코딩한 값을 반환하는 구현체
  - 의존성 주입으로 테스트객체에 주입시켜 필요한 로직 검증에만 집중할 수 있다.
  ```java
    Http http = (String url) -> "{json Result..}"
  ```
- mock : 의도적으로 흉내낸 동작을 제공하고, 수신한 인자가 정상인지 여부를 검증하는 일을 하는 테스트 구조물. (stub에서 자체 검증 추가)
- mock 은 프로덕션코드를 직접 테스트하고있지않으므로, 단위테스트 커버리지의 구멍을 만든다. 통합테스트도 작성하자.
# 11 테스트 리팩토링
- 테스트코드에서 try/catch - fail은 의미가 없다. 오류를 던지면 알아서 실패, 로깅해준다.
- 테스트 코드에서 변수를 역으로 참조하기전에 null체크는 의미가 없다.
  ```java
  List<String> strings = new ArrayList<>();
  assertThat(strings).isNotNull();    // 아래서 오류를 던질것이므로, 불필요한 검증문.
  assertThat(strings).hasSize(1);
  ```
- 좋은 테스트는 클라이언트가 시스템과 어떻게 상호작용하는지 추상화한다.
- Matcher를 사용자정의로 구현하려면, 더많은 코드가 필요하지만 테스트 가독성이 좋아지므로 충분히 가치있다.
  - 단일 개념을 구현하는 2~3줄의 코드가 있다면 한줄로 개선하라
- 테스트 한개에는 assert 한개만
- 테스트와 무관한 세부사항은 private 메서드 혹은 @Before/@After 로 옮기자. **테스트를 이해하는데 필요한 정보는 제거하지 않아야한다.**
- 
# 12 TDD
1. 실패하는 테스트 코드 작성
2. 테스트 통과시키기
   1. 테스트의 명세를 나타내고 통과할 수 있는 수준의 코드만 추가하라.
   2. 커밋 사이즈를 작게하여 백업에 용이하게 하자.
3. 이전 두 단계에서 추가되거나 변경된 코드 개선하기
4. 테스트 정리
   1. 중복되는 초기화 코드를 `@Before` 메서드로 옮겨 중복코드를 줄이자.
   2. TDD는 거의 모든 코드에 안전한 리팩토링을 가능하게 한다.
5. 테스트 증분 -> 프로덕션코드 증분
   1. TDD를 성공하려면, 시나리오를 테스트로 만들고 각 테스트를 통과하게 만드는 코드 증분을 최소화하는 순으로 코드를 작성하는 것.
6. 테스트 이름
   1. 중복되는 테스트명을 NestedClass로 테스트 클래스를 분리하면 메서드 명을 깔끔하게 유지 할 수 있다.

# 13 까다로운 테스트
## 동시성 테스트
- 동시성 처리가 필요한 테스트는 단위테스트보다 통합테스트에 가깝다.
- 스레드 통제와 애플리케이션 코드 사이의 중첩을 최소화하라.
  - 스레드없이 단위테스트할 수 있도록 설계를 변경하여, 남은 최소한의 작업만 스레드 테스트하라.
- java에서 제공하는 기본 클래스 자체를 테스트하지는 말자. 이미 많이 입증된 것!
- 예시코드
- Production code
  ```java
  class Matcher {
    private ExecutorService executor = Executors.newFixedThreadPool(10);

    public void find(MatchListener listener, List<MatchSet> matchSets, BiConsumer<MatchListener,MatchSet> processFunction){
      //...
      foreach(var set : matchSets){
          Runnable runnable = () -> processFunction(listener, set);
          executor.execute(runnable);
      }
      executor.shutdown();
    }
  }
  ```
    - 멀티스레드로 처리되는 부분만 코드로 분리되어있으며, 상세한 동작들은 파라미터로 받도록 되어있다.
  - test code
    ```java
    @Test
    void test(){
      Set<String> processedSets = Collections.syncronizedSet(new HashSet<>());
      
      matcher.find(...,(listener,set) -> processedSets.add(set.getId()))
      while(!matcher.getExecutor().isTerminated());
      
      assertThat(processedSets).contains(...);
    }
    ```
    - executor를 노출시키고 loop문으로 멀티스레드 작업의 완료를 기다린다.
    - listener를 파라미터로 받게하여 결과를 테스트가능하게 한다.

## db 테스트
- 테스트 안에서 데이터를 생성하고 관리해아한다.
- 테스트마다 깨끗한 데이터베이스로 시작히야한다.
  - 대안 1 : `@Before` , `@After` 에서 deleteAll 하라
  - 테스트마다 db초기화가 어렵다면, 테스트마다 트랜잭션을 초기화하여 테스트가 끝날때 롤백한다.
- db 테스트는 따로 문리하여, 통합테스트의 갯수와 복잡도를 최소화하고, 단위테스트에서 검증하는 로직을 최대화 해야 한다.
- 느리거나 휘발적인 코드를 목으로 대체하여 단위 테스트의 의존성을 끊어야한다.

# 14. 프로젝트에서 테스트
- 단위테스트가 팀 문화가 될 수 있도록 빠르게 도입을 시도해보자.
- 팀원들과의 의견과 시선이 맞야한다. 같이 공부하고, 토론하여 합의점을 늘려가자.
- 단위테스트 표준을 만들자. 적어도 분기별로는 다시 살피고 고쳐야한다.
  - 코드를 체크인 하기전에 어떤 테스트를 실행할지 여부
  - 테스트 클래스와 메서드의 이름 짓는 방식
  - 검증문, mock 라이브러리 선택
  - AAA 사용 여부
- PR등으로 코드리뷰를 하여 표준을 준수하도록 장려한다.
  - 책에서 소개하는 intellJ upsource는 2022.2 부터 유지보수만 지원중.
- 커버리지 개념은 오로지 속임수를 써야만 100%에 도달할 수 있다는 제한이 내재되어있다.
- 코드를 작성하고 습관적으로 단위테스트를 작성하는 팀들은 비교적 쉽게 70%의 커버리지를 달성한다.


----
- [code](https://github.com/gilbutITbook/006814)
- [hamcrest 2.2 java](https://hamcrest.org/JavaHamcrest/javadoc/2.2/)