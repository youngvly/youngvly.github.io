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
    - 

----
- [code](https://github.com/gilbutITbook/006814)
- [hamcrest 2.2 java](https://hamcrest.org/JavaHamcrest/javadoc/2.2/)