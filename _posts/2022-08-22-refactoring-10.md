---
---
layout: post
title: "Refactoring 10 조건부 로직 간소화"
date: 2022-08-22 23:51
categories: Refactoring
tags: cleanCode, study
---
# 10 조건부 로직 간소화

## 10.1 [조건문 분해하기](https://refactoring.com/catalog/decomposeConditional.html)
- 조건을 검사하고 무슨일이 일어나는지는 나오지만, 왜 일어나는지는 설명되지 않는다.
- 의미부여된 이름의 함수호출로 바꾸면 의도를 쉽게파악할 수 있다.
- 조건문이 간단해지면 삼항연산자로 바꾸기도 가능.


## 10.2 [조건식 통합하기](https://refactoring.com/catalog/consolidateConditionalExpression.html)
- 조건은 같지만 결과가 같을때 조건검사를 하나로 통합하자.
  - 하려는 일이 명확해진다.
  - 함수추출까지 이어질 가능성이 높다.
- and, or이 복합적으로 나타나는 코드라면 적당히 나눠서 함수추출하기로 의미부여하자.
### 절차
1. 조건식들 모두에 부수효과가 없는지 확인한다.
2. 조건문 두개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트 & 반복


## 10.3 [중첩 조건문을 보호 구문으로 바꾸기](https://refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html)
- 보호구문(guard clause) : if 에서는 정상 동작, else에서 return
### 절차
1. 교체해야할 조건중 바깥것을 선택하여 보호구문으로 바꾼다.
2. 테스트 & 반복

## 10.4 [조건부 로직을 다형성으로 바꾸기](https://refactoring.com/catalog/replaceConditionalWithPolymorphism.html)
- 타입을 기준으로 분기하는 switch문이 포함된 함수가 여러개 보인다면!
  - 기본동작은 슈퍼클래스로
  - 변형동작은 서브클래스로
- 거의 같은 객체이지만, 다른 부분도 있음을 명확하게 표현하기위해 다형성으로 표현하기도 한다.
### 절차
1. 다형성 클래스와 기왕이면 팩토리까지 만든다.
2. 호출하는 코드에서 팩토리함수를 호출한다.
3. 조건부로직 함수를 슈퍼클래스로 옮긴다.

## 10.5 [특이케이스 추가하기](https://refactoring.com/catalog/introduceSpecialCase.html)
- 특정 값에 대해 똑같이 반응하는 코드가 여러곳이라면 반응을 특이케이스객체로 모으자.
- null 객체 패턴
- 데이터구조를 읽는 동작만 있다면(get) 리터럴객체 사용 (map)
- 변환함수 이용하기
  > 원본 데이터구조에서 clone되긴했지만 나중에 변경위치를 찾는데 애먹을것. 안좋은패턴같아보임.
### 절차
1. 특이케이스인지 검사하는 속성을 추가하고 false를 반환하게한다.
   ```java
   class Customer {
    boolean isUnknown() {return false}
   }
   ```
2. 특이케이스 객체를 만든다. 위 속성만 포함하고 이 속성은 true를 반환하게한다.
  ```java
  class UnknownCustomer {
    boolean isUnknown() {return true}
  }
   ```
3. 클라이언트에서 특이케이스 검사하는 함수를 추출한다. 모든 클라이언트가 값을 직접 비교하는 대신 함수를 호출한다.
   ```java
    boolean isUnknown(String name){
      if(name == "미확인고객") {return true}
      throw new Error("잘못된 비교, 리팩토링 실수방지용 에러")
    }
   ```
4. 코드에 새로운 특이케이스 대상을 추가한다. 함수의 반환값으로 받거나 변환함수를 적용하면 된다.
  ```java
    if (this.customer == "미확인고객") {
      return new UnknownCustomer();
    }
  ```
5. 특이케이스를 검사하는 함수(3) 본문을 수정하여 객체 속성을 사용하도록 한다. -> 잘 적용되었는지 확인용.
  ```java
    boolean isUnknown(Customer arg){
      if(! arg instanceof UnknownCustomer) {return true}
      throw new Error("잘못된 비교, 리팩토링 실수방지용 에러")
    }
  ```
6. 테스트
7. 여러 함수를 클래스로 묶기나 변환함수로 묶어 특이케이스를 처리하는 공통동작을 새로운 요소로 옮긴다.
  - UnknownCustomer 에 메서드, 필드로 특이케이스의 반응(동작)을 옮긴다.
  - 특이케이스는 불변,VO이어야 한다.
8. 특이케이스 검사 함수를 이용하는 곳이 남아있다면 검사함수를 인라인한다.

## 10.6 [assertion 추가하기](https://refactoring.com/catalog/introduceAssertion.html)
- assertion이 실패했다는것은 개발자가 잘못했다는 것.
- assertion의 실패는 절대 검사하지않아야한다.
> validation체크로 사용하기도 했던것같은데 여기서 말하는 assertion의 용도가 맞는듯. 근데 서비스 코드에 같이 딸려들어가서 배포까지 나가는게 맞는것인지?
- 단위테스트를 꾸준히 추가하여 사각을 좁히면 assertion보다 나을때가 많다.
- 반드시 참이어야하는것만 검사한다.

## 10.7 [제어플래그를 탈출문으로 바꾸기](https://refactoring.com/catalog/replaceControlFlagWithBreak.html)
- 제어플래그 : 코드의 동작을 변경하는데 사용되는 변수
### 절차
1. 제어플래그 사용부를 함수로 추출가능한지 고려
2. 제어플래그를 갱신하는 코드를 각 적절한 제어문으로(return,break,continue) 바꾸고 테스트
3. 제어플래그 제거
