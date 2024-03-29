---
title: "TDD 3장 테스트주도 개발의 패턴"
date: 2023-01-30 19:10:00 +0400
categories: test, study
tags: test, tdd
---

# 25. 테스트주도 개발 패턴
### 격리된 테스트
- 테스트들은 실행순서와 결과 등에 서로 독립적이어야한다.
### 테스트 목록
- 할일 목록을 나열하고 우선순위를 정하라. 할일 목록을 한번에 테스트로 구현하는것은, 나중에 리팩토링에 관성을 가지게되어 걸림돌이 될 수 있고, 전체 테스트가 통과하게 하기까지 많은 시간이 소요되므로, 하나씩 정복하는것이 좋을것이다.
### 테스트를 먼저 작성
- 테스트의 대상이 되는 코드를 작성하기 전에 테스트를 먼저 작성하는것이 좋다.
- 테스트는 프로그램 설계와 작업 범위 조절에 유용하다. 
### assert 우선
- assert를 먼저 작성하면 작업을 단순하게 만들 수 있다.  => 결과를 얻기위해 필요한것들을 역으로 유추하는 방식.
### 테스트 데이터
- 데이터 간의 차이가 있다면 그 속에 의미가 있어야한다. 1과 2사이에 개념적 차이가 없다면 1을 사용하자.
- 테스트 데이터에 대한 대안은, 실제 세상에서 얻어진 실제 데이터를 사용하는것.
### 명백한 데이터
- 테스트 자체에 예상되는 값과 실제값을 포함하여 둘사이의 관계를 드러내야한다.
- 기호상수가 이미 있다면 사용하라.
```java
  // asis
  assertEquals(4.95 result);
  // tobe
  assertEquals(100 / 2 * (1 - 0.015), result)
```
> 단위테스트였나, 결과값은 테스트안에서 계산되지않고, 값으로 비교되어야한다고 본것같은데,, 사실 구현내용과 동일한 계산로직을 테스트 결과값에서도 동일하게 사용된다면 의미있는 테스트가 아닐것같다. 

# 26. 빨간 막대 패턴
### 한단계 테스트
- 목록중에서 다음 테스트를 선정할때, 구현할 수 있다고 확신이 드는것을 선정하라. (아는것에서 모르는것으로 구현하여 성장하라.)
### 시작 테스트
- 간단한 오퍼레이션부터 테스트를 시작하라.
  - 출력이 입력과 같은경우?
### 설명 테스트
- 테스트로 설명을 요청하고 설명하라.
### 학습 테스트
- 외부 라이브러리를 학습하기위해 테스트를 작성해볼 수 있다.
### 또 다른 테스트
- 회의중에 주제에서 벗어나지 않으려면, 주제와 벗어나는 아이디어가 떠올랐을때 테스트를 할일 목록에 적고 다시 주제로 돌아오자.
### 회귀(regression) 테스트
- 장애가 발생했을때, 장애로 인하여 실패하고 복구시 통과될수있는 테스트를 작성하라.
- 전체 애플리케이션 차원에서 테스트를 수행하는것도 가치가있다.
### 휴식
- 피로는 판단력에 음성적인 영향을 끼치고, 판단력은 다시 피로에 음성적인 영향을 끼친다.
- 휴식 주기를 알아차리기위한 방식
  - 물병을 비우면 휴식하자
  - 정규 근무시간 후의 약속이 진행을 멈추는데에 도움이 될 수 있다.
  - 주단위로는 주말활동
  - 년단위로는 강제 휴가정책 회복1주~복귀1주 => 최소 3주~4주
### 다시하기
- 코드가 뒤죽박죽이되어 갈길을 잃었을땐, 다지우고 처음부터
### 싸구려책상 좋은 의자

# 27. 테스팅 패턴
### 자식 테스트
- 큰 테스트케이스를 돌아가게하려면, 깨지는 부분을 작은 테스트로 분리하고, 성공하면 큰테스트를 추가하라.
- 왜 큰 테스트가 나올수밖에 없었는지 돌아보자. 작게 여러개로 쪼갤 수 있다면 큰 테스트를 삭제하자.
### 모의 객체 (Mock object)
- 테스트할때 db는 의존성이 높기때문에, mock db를 사용할 수 있다.
- mock object는 테스트안에 기대결과값을 확인할수있어 가독성이 높다.
### self shunt = loop back
- 한 객체가 다른 객체와 올바르게 대화하는지 테스트하기위해, 테스트 대상 객체를 테스트 클래스로 만들면된다.
- 테스트 대상 클래스의 인터페이스 추출이 필요하다. 클래스를 블랙박스 테스트하는것이 좋은지, 인터페이스 추출이 쉬운지는 판단이 필요.
```python
  # test target interface
  class ResultListener :
      def __init__(self) :
        self.count = 0
      def startTest(self)

  # test class
  class ResultListenerTest(ResultListener) :
    # TestResult 와 listener 간의 통신을 테스트
    def testNotification(self) :
      self.count = 0
      result = TestResult()
      result.addListener(self)    # listener 객체를 따로 만들지않고, 테스트케이스가 listener가 되어 테스트한다.

      WasRun("testMethod").run(result)

      assert 1 == self.count
    

    def startTest(self):
      self.count += 1
```
### 로그 문자열
- 오퍼레이션의 호출순서등을 확인하기위해 문자열에 순차적으로 저장하는 로그 문자열을 사용할 수 있다. `assert("setUp testMethod tearDown" == result.log)`
- 자바에서는 `List<String>` 이 낫겠다.
### 크래시 테스트 더미
- 재연하기힘든 에러상황을 테스트해야한다면, 재연 상황을 구현하는 대신 그냥 예외를 발생시키는 객체를 만들면 된다. (override)
```java
  public void testFileSystemError(){
    File f = new File("foo"){
        @Override
        public boolean createNewFile() throws IOException {
          throw new IOException();
        }
      } 
    }

    assertThrow(() -> saveAs(f), IOException.class)
  }
```
### 깨진 테스트
- 혼자서 프로그래밍할때 프로그래밍 세션을 끝마칠때는 테스트가 깨진 상태로 끝마치는것이 좋다.
- 나중에 돌아왔을때 어느 작업부터 시작할 것인지 명백히 알 수 있다.

### 깨끗한 체크인
- 팀 프로그래밍을 할때 프로그래밍 세션을 끝마칠때는 모든 테스트가 성공한 상태로 끝마치는것이 좋다.
- 팀원들이 코드를 체크인하기전에, 모든 테스트가 돌아간다는것이 보장되어야한다.
- 실패한 테스트는 방금 본인이 짠 프로그램을 완벽히 이해하지 못한다는 뜻이다. 깨지면 다 날리고 다시하는것도 방법이다.

# 28. 초록 막대 패턴
- 깨지는 테스트를 가능한 빨리 통과하게 만들어라.
## 가짜로 구현하기
- 실패하는 테스트 구현 후 첫번째 구현은 상수 리턴으로 구현할 수 있다.
- 심리적으로 테스트가 성공한다는 안정감을 얻을 수 있고
- 실제 구현할때에도, 이전 테스트의 작동이 보장되기 때문에 범위조절에 용이하다.
```java
@Test
void testSum(){
  assertThat(plus(1,3)).isEqualTo(4)
}

// production
int plus(int augend, int addend){
  return 4;
}
```
 
## 삼각 측량
- 검증부가 2개 이상 있을때만 추상화하라.
```java
@Test
void testSum(){
  assertThat(plus(3,1)).isEqualTo(4)
  assertThat(plus(3,2)).isEqualTo(5)
}

// production
int plus(int augend, int addend){
  return augend + addend;
}
```
- 삼각측량법 적용 후 assert문이 중복이므로 assert문을 하나만 남기고 삭제하면, 프로덕션 코드에서 상수반환으로 수정될수있고, 이는 무한루프를 일으킬 수 있다. 구현 이후에 리팩토링된다면 해결되는문제가 아닌가?
- 필자는 추상화에 감이 안올때에만 삼각측량을 사용한다고 한다.

## 명백한 구현
- 단순한 연산은 그냥 구현해버리자.
- 제대로 동작하는것과 깨끗한 코드를 한번에 만족시키는것은 많은 일 일수 있으니, 리팩토링을 나중에 하는것으로 해결하자.
## 하나에서 여럿으로
- 컬렉션을 다루는 연산을 구현할때에는, 일단 컬렉션없이 구현해보고 다음에 컬렉션 구현으로 변화시켜보자.
```java
@Test
void testSum(){
  assertEquals(8, sum(5,1,2))
}

private int sum(int ...values){
  ...
}
```   

# xUnit 패턴
## 단언 assert
- 프로그램이 자동으로 코드가 동작하는지에 대한 판단을 하도록 하라.
- 판단 결과가 boolean 값이어야한다. (true는 성공/ false는 실패)
- assert는 구체적이어야한다. 
  - 나쁜 예  : `assert( rectangle.area() != 0 )`
  - 개선 예 : `assert (rectangle.area == 50)`
- public 인터페이스만 테스트로 검증해야한다. 화이트박스 검증이 필요한건 설계의 문제이다.

## fixture 
- 여러 테스트에서 공통으로 사용하는 객체 생성할때
- 객체를 세팅하는 코드는 중복인 경우가 많아 이런 객체를 test fixture 라 한다.
- fixture 구현법
  - instance field로 분리 후, setUp method 에서 초기화 > 초기화 방색을 기억해야한다.
  - 테스트 메서드내에서 중복 구현 > 중복 코드를 감안해야 하지만 가독성은 좋다
  - 객체 생성을 메서드로 꺼내서 호출 > 위 두 단점을 보완할 수 있지 않을까 
- 다른 fixture가 필요한 경우, 필자는 이너클래스로 분리하여 setUp 메서드를 분리한다.  
- 
## 외부 fixture
- file과 같은 외부 자원이 있는경우, 객체가 인스턴스 변수로 꺼내져있다면, tearDown 과정에서 release 해줄 수 있다. (테스트 메서드마다 finally 중복X
## 테스트 메서드
- 동일한 픽스처를 공유하는 모든 테스트는 동일한 클래스의 메서드로 작성될 수 있다.
- 메서드 이름은 왜 테스트가 작성되었는지를 나타내야한다.
- 테스트 작성전에 원하는 테스트 동작 목록을 주석으로 적어두면,테스트 클래스에 어떤 테스트가 필요할지 정리할 수 있다.
## 예외 테스트
- 예외가 발생하는것이 기대동작일 경우, 예외가 발생하지않는경우 테스트가 실패하게 하면 된다.
- Rule 이용, assertThrown(..) , @Test(expected = ..Exception.class) 등등
## 전체 테스트
- 테스트 슈트에 대한 모음을 작성하면 된다. 
```java
  public static Test suite(){
    TestSuite result = new TestSuite("test 모음")
      result.addTestSuite(MoneyTest.class)
      result.addTestSuite(ExchangeTest.class)
      return result;
  }
```  
- intellij에서는 패키지 단위로 돌리면 된다. 

# 30. 디자인 패턴
- 이 책에서는 리팩토링을 설계의 일종이라 보지않고, 설계와 디자인패턴을 다르게 본다.
## Command : 계산 작업에 대한 호출을 메시지가 아닌 객체로 표현한다.
  - 단순 메서드 호출보다 복잡한 계산은 계산을 위한 객체를 생성하자. ex) Runnable
## VO : 객체가 생성된 이후 값이 절대 변하지 않게 하여 참조 문제가 발생되지 않게한다.
  - 객체가 널리 공유되어야하지만, 동일성은 중요하지 않을때
  - 참조에 의한 의도치않은 값변경 문제 해결방안
    -  객체의 참조를 공유하지않고, 복사객체를 가지게 하는 방법 > 메모리 낭비, 공유객체의 상태변화 공유불가
    - observer : 객체의 상태가 변화되면 통지를 받는 방법 > 흐름 이해가 어렵고, 로직이 지저분해질 수 있다.
    - VO : 애초에 객체가 변화하지않는다면 참조 문제가 발생되지않는다. > 성능문제는 문제가 생기면 고민하자.
## Null Object : 계산 작업의 기본 사례를 객체로 표현한다
  - 동일한 인터페이스를 구현하는 null 상황을 표현하는 객체를 만들어 중복되는 null 검사를 없앨 수 있다.
## Template Method : 계산작업의 변하지 않는 순서를 여러 추상 메서드로 표현하고, 상속을 통해 구체화.
  - 작업 순서는 변하지않지만, 구현이 변할 가능성이 있는경우 적용
  - 초기 설계보다는 리팩토링단계에서 발견되는게 좋을것.
## Plugable Object : 둘 이상의 구현을 객체를 호출함으로써 다양성을표현한다
- 조건문 중복을 해결하기위해 인터페이스 추출후 인터페이스 객체를 호출한다.
```java
// ASIS
class SelectionTool {
  Figure selected;
  void mouseDown(){
    selected = findFigure()
    if (selected != null) {
      select(selected)
    }
  }
  void mouseMove(){
  if (selected != null){
    move(selected);
  else
    moveSelectionRectengle();
  }
  ..  
}

// TOBE
class SelectionTool {
  SelectionMode mode;
  void mouseDown(){
    selected = findFigure()
    if (selected != null) 
      mode = SingleSelection(selected)
    else
      mode = MultipleSelection()
  }
  void mouseMove(){
    mode.move()
   }
   .. 
```
## Plugable Selector : 객체별로 서로 다른 메서드가 동적으로 호출되게 함으로써 필요없는 하위 클래스 생성을 피한다.
- 인스턴스별로 서로 다른 메서드가 동적으로 호출되게 하는 방법
  - 하나의 오퍼레이션을 가지는 인터페이스를 구현하는 하위 클래스가 여러개라면 상속은 무거울수있다.
    ```java
      interface Report {
        void print();
      }
      class HTMLReport implements Report {
        void print() {...}
      }
      class XMLReport implements Report {
        void print() {...}
      }
    ```
  - switch문을 가지는 하나의 클래스를 만들어 호출 > 오퍼레이션명이 여러군데 중복되어 흩어진다.
  ```java
  class Report {
    final String printMessage;
    void print(){
      switch(printMessage) {
        case "printHTML" : printHTML(); break;
        case "printXML" : printXML(); break;
        ..
      }
    }
    void printHTML();
  }
  ```
  - plugable selector : reflection으로 호출한다. 메서드를 하나가지는 하위클래스가 한뭉치 존재할때와 같은 직관적인 상황에서 코드를 정리하기 위한 용도로만 사용되어야 한다.
  ```java
  class Report {
    final String printMessage;
    void print(){
      this.getClass().getMethod(printMessage, null).invoke(this,new Class[0]);
    }
    void printHTML(){}
    
  ```
  > 구현 코드와 public 인터페이스의 강결합아닐까'ㅂ'
## Factory Method
- java에서 생성자 사용은 표현력과 유연함이 떨어진다.
- 객체의 유연함이 필요할때에만 사용하자. (상위 객체의 생성자에서 다른 클래스 객체를 생성할 수 있게 하는 것)
## Imposter : 인터페이스를 구현하는 객체를 추가하여, 시스템에 변이를 도입한다.
- null 객체
- composite
## Composite : 하나의 객체로 여러 객체의(collection) 행위 조합을 표현한다.
- 객체 집합을 나타내는 객체를 단일 객체에 대한 imposter로 구현한다. 
```java
interface Holding (소유재산) {
  Money balance();
}
  
class Transaction implements Holding {
  Money value;
  Money balance() return value; 
}
class Account implements Holding {
  Holding holdings[];
  Money balance() {
    ...
    return sum
  }
```   
- Account(계좌)의 잔액(balance)은 Transaction(거래내역)의 합으로 구성된다.
- 여러 계좌의(Account) 잔액(balance)은 계좌의(Account) 합으로 구성된다.
  - 이때 여러 계좌라는 클래스를 뽑지않고, Account 자신을 동일한 인터페이스를 가지도록 하였다.
- Account는 소유재산의 합을 잔액으로 보여준다.
## 수집 매개 변수 : 여러 다른 객체에서 계산한 결과를 모으기 위해 매개변수를 여러곳으로 전달한다.
## Singleton : 전역변수를 제공하지 않는 언어에서는 전역변수를 사용하지 마라.


# 31. 리팩토링

## 차이점 일치시키기
- 비슷해보이는 두 코드조각을 합치려면, 단계적으로 닮아가게끔 수정하고, 동일해지면 합친다.
## 변화 격리하기
- 객체나 메서드의 일부만 바꾸려면, 일부를 격리하고 수정하라. 작업을 되돌리기 쉽고, 수정부분만 집중할 수 있다.
## 데이터 이주시키기
-  내부에서 외부로 표현양식을 변경시키려면, 일시적으로 데이터를 중복시켜야한다.
```python
//  asis 1개의 테스트만 
class TestSuite:
  def add(self, test):
    self.test = test
  def run(self, result):
    self.test.run(result)
    
// tobe 여러개의 테스트를 모으는 testSuite
class TestSuite :
  self.tests = []
  
  def add(self,test) :
    self.test = test    //일시적 중복, 추후 삭제
    self.tests.append(test)
  def run(self, result):
    self.test.run(result)    //일시적 중복, 추후 삭제
    for test in self.tests:
      test.run(result)
``` 

## 메서드 추출하기
- 메서드 추출하기 자동화툴을 사용할수도 있고, 반복문이나 중복코드, 내용 설명을 위한 주석이 필요한 곳에서 메서드 추출을 사용할수있을것.
## 메서드 인라인.
- 리팩토링중 추상화 계층 이해가 어렵다면, 인라인시켜 흐름을 이해한 뒤 리팩토링하는것도 방법.
## 인터페이스 추출하기
- 동일한 오퍼레이션에 대한 두번째 구현을 하고싶을때, 인터페이스로 추출할수있다.
## 메서드 옮기기
- 다른 객체에 대한 두개이상의 오퍼레이션을 호출할때 시도해볼 수 있다.
- 메서드 일부만 옮기고싶을땐, 메서드 추출하기 > 메서드 옮기기 > 기존 코드 수정하기.
## 메서드 객체
1. 메서드와 같은 매개변수를 갖는 객체를 만든다.
2. 메서드의 지역변수를 객체의 인스턴스 변수로 만든다.
3. 원래 메서드와 동일한 내용을 갖는 이름의 메서드를 만들고, 그 메서드를 호출한다. 
- 메서드 추출하기를 적용할 수 없는 코드를 간결하게 만들기 위한 용도로 적합하다. 
  - 여러 임시변수와 매개변수로 얽혀있는경우 객체가 역할을 대신할 수 있다.
## 매개변수 추가 
- 매개변수 추가, 컴파일에러나는 부분 수정
## 메서드 매개변수를 생성자 매개변수로 바꾸기
- 동일 매개변수를 여러 메서드에서 사용하는경우, 생성자에서 인스턴스 변수로 올려 this.xx를 사용하게 하고, 메서드 매개변수를 없앨 수 있다.

# 32. TDD 마스터하기
- 한 단계는 얼마나 커야 적합한가 
  - 작게든 크게든 할수있어야하지만, 리팩토링 초기에는 단계가 작아야한다.
- 테스트할 필요가 없는것은?
  -  조건문/반복문/연산자/다형성 에 대해 본인이 작성한 코드에서만 테스트하라
- 좋은 테스트인지 어떻게 확인할까
   - 설계 문제를 알려주는 지표 
     - 긴 setup (객체가 크다, 쪼개자)
     - setup중복 (공통 setup으로 분리할 수 없다면, 객체들이 밀접하게 얽혀있다는것)
     - 시간이 오래걸리는 테스트 (테스트를 자주 돌리지 않게 된다)
     - 깨지기 쉬운 테스트 (객체간의 의존도가 높다는것)
- TDD로 프레임워크를 만들려면?
  - TDD는 발생하지않은 변수는 커버하지못하지만,예상가능한 변수들을 잘 표현하는 프레임워크를 만들수있다.
- 피드백이 얼마나 필요한가
  - 실패율을 얼마나 허용할지는 개인의 선택, 여기서는 MTBF(Mean Time Between Failure) 실패시점간의 시간 (하루에 한번꼴 = MTBF 24h)
- 지워야하는 테스트?
  - 테스트를 삭제했을때 코드 안정성에 자신감이 떨어진다면 그냥 두자.
  - 테스트 코드가 동일한 부분을 실행하더라도, 서로 다른 시나리오를 말한다면 그냥두자.
- 프로그래밍 언어나 환경이 TDD에 어떤 영향을 주는가
  - 언어 및 환경에 따라 TDD주기(테스트/컴파일/실행/리팩토링)이 길어지면 단계가 커지는 경향이 있다.
- 거대한 앱을 만들때도 TDD적용이 가능할까?
  - YES, 크기는 TDD 효율과 무관하다. 오히려 중복제거 > 작은 객체 > 쉬운테스트 로 설계가 개선될것.
- 애플리케이션 수준의 테스트로도 개발을 주도할 수 있는가?(ATDD)
  - 작은 테스트는 (단위테스트?), 개발자가 예상가능한 시나리오로 구현하여 위험성이 있다.
  - ATDD는 사용자가 테스트를 주도하여, 사용자의 책임이 생기며, 협조가 필요하다.
  - 위 같은 이유로 피드백이 늦어져 빨강>초록>리팩토링 이 어려울 수 있다.
  - TDD가 어려울수있어 비추
- 프로젝트 중반에 TDD를 도입할 수 있을까
  - 테스트를 고려하여 구현된 코드가 아니기때문에, 당장 테스트를 작성하기 어려움.
  - 테스트 추가를 위해 리팩토링이 필요하지만, 테스트가 없기때문에, 리팩토링 안정성을 보장할 수없다. (닭이먼저냐 달걀이먼저냐 문제와 비슷)
  - 조심스럽게 작업하거나 다른 방법으로 테스트를 대신할 피드백을 얻어서 리팩토링/테스트 추가를 하는 방법을 사용할 수 있다.
- TDD는 누구를 위한 것인가
  - 장기적으로 프로젝트를 이끌어가길 원하는 개발자, 테스트가 있으면 안전하게 리팩토링이 가능하기때문에, 개발에 자신감을 얻을 수 있다.
- TDD와 패턴의 관계는?
  - 패턴을 적용할수있는 구간에서 적용한다면, 빠르게 코딩이 가능하며, 이는 패턴을 적용할 수 없는 구간에서의 시간을 벌어준다.
- 왜 TDD가 효과적인가
  - TDD는 결함을 빠르게 발견할 수 있게 하여, 결함 수정에 드는 비용이 줄어들고, 결함도 줄어들 수 있다. 
  - 설계 결정에 대한 피드백을 빠르게 받을 수 있다. > 인터페이스를 원하는대로 수정하고, 동작에 대한 피드백을 테스트로 받을 수 있다.
- TDD와 XP사이의 관계는?
  - 짝프로그래밍 : 테스트는 짝과의 좋은 의사소통 수단이 된다.
  - 활기차게 일하기 : 테스트별로 주기를 나눌 수 있어서, 힘들면 다음 테스트를 구현하기전에 쉬어갈 수 있다.
  - 지속적인 통합 : 테스트는 지속적인 통합이 안전하게 가능하도록 해준다.
  - 단순 설계 : 테스트로 구현 범위를 정해놨기 때문에, 요구사항에 맞는 양만큼만 구현하게 해준다.
  - 리팩토링 : 테스트가 있으면 안전하게 리팩토링이 가능하다.
  - 지속적인 배포 : 테스트가 있으면 적게 수정하고 안전하게 자주 배포할 수있다.