---
---
layout: post
title: "Refactoring 09 데이터 조직화"
date: 2022-08-22 23:51
categories: Refactoring
tags: cleanCode, study
---
# 09 데이터 조직화

## 9.1 [변수 쪼개기](https://refactoring.com/catalog/splitVariable.html)
- 대입이 두번이상 이뤄진다면 여러 역할을 수행한다는 신호. 역할 하나당 변수 하나를 사용하자.
- 입력 매개변수의 값이 수정될때
### 절차
1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 바꾼다.
   1. 수집변수는 제외 (ex) for문의 i)
2. 가능하면 불변으로 선언.
3. 두번째 대입전까지 변수명을 수정한다.
4. 두번째 대입시 변수를 원래이름으로 다시 선언한다.
5. 테스트 , 반복


## 9.2 [필드 이름 바꾸기](https://refactoring.com/catalog/renameField.html)
### 절차
1. 레코드의 유효범위가 제한적이라면 모든 코드를 수정하고 테스트.
2. 레코드가 캡슐화되지 않았다면 레코드를 캡슐화부터(7.1)
3. private 필드명 변경하고 내부 메서드 수정
4. 테스트


## 9.3 [파생변수를 질의함수로 바꾸기](https://refactoring.com/catalog/replaceDerivedVariableWithQuery.html)
- 값을 쉽게 계산할 수 있는 변수들을 모두 제거할 수 있다.
- 계산 과정을 보여줌으로써 데이터 의미를 쉽게 파악할 수 있다.
- 파생변수의 변경을 누락하는 일이 없다.
- getCalc() {return a - b} 에서 a,b가 불변이라면 파생변수가 낫다. => c = a-b
- 새로운 데이터 구조를 생성하는 변형연산이라면 그대로 두자.
  - 변형연산?
    - 데이터 구조를 감싸며(?), 그 데이터에 기초하여 계산한 결과를 속성으로 제공하는 객체
    - 데이터 구조를 받아 다른 데이터 구조로 변환해 반환하는 함수
      - 소스데이터가 가변이고, 파생데이터 구조의 수명을 관리해야할때는 객체를 사용하는편이 유리
      - 소스데이터가 불변이거나 파생데이터를 잠시쓰고 버린다면 어느방식을 써도 좋다.
    ```java
        // 생각나는대로 예시코드
        // 객체사용이 유리
        private int a;
        private int b;
        private NewClass c;
        public void setA(int a) {
          if (this.a == 0){
            this.c = null; //수명관리 ???
            return;
          }
          this.c = NewClass.builder().field(a-b).build();
        }
        
        // ----------------
        // 질의함수 사용가능
        private final int a;
        private final int b;
        public OldClass(int a, int b){
          this.a = a;
          this.b = b
          this.new = NewClass.builder().field(a-b).build();
        }
        // 질의함수 사용으로 변경
        public NewClass getNew(){
            return NewClass.builder().field(a-b).build();
        }
    ```
      
### 절차
1. 변수 값이 갱신되는 지점을 모두 찾고, 필요하면 변수쪼개기로 갱신지점에서 변수 분리.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든곳에 assertion을 추가하여 계산결과가 변수값과 같은지 확인.
  ```js
    get production(){
      assert(this._production === this.calculatedProduction); // 함수호출과 값이 동일한지 확인
      return this._production //기존 코드, 매번 계산되던 변수
    }
    get calculatedProduction(){ // 추가된 메서드
      return this._adjustments.reduce((sum,a) => sum + a.amount,0);
    }
  ```
4. 테스트
5. 변수를 읽는 코드를 모두 함수 호출로 대체
6. 테스트
7. 변수를 선언하고 갱신하는 코드를 삭제.
    
## 9.4 [참조를 값으로 바꾸기](https://refactoring.com/catalog/changeReferenceToValue.html)
- 반대 : 값을 참조로 바꾸기 (9.5)
- 데이터구조가 다른 데이터구조를 품는 경우
- 참조로 다루는경우 에는 내부객체는 그대로 둔채, 그 객체의 속성만 변경  `set officeAreaCode(arc) {this._telephone.areaCode = arc}`
- 값으로 다루는경우, 새로운 속성을 담은 객체로 기존 내부객체를 통채로 대체. (내부객체 -> VO) `set OfficeAreaCode(arc) {this._telephone = new Tel(arc,...)}`
### 절차
1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인
2. 불변클래스로 만들기 = setter를 제거
3. VO 필드를 사용하는 equals 메서드를 만든다 => 진짜 VO

## 9.5 [값을 참조로 바꾸기](https://refactoring.com/catalog/changeValueToReference.html)
- 반대 : 참조를 값으로 바꾸기 (9.4)
- ex) A:B = 1:n 구조일때, A가 VO로 n번 복사해서 B에 들어간다면, A가 수정될때 데이터 일관성이 깨질 수 있다. 참조를 사용해야한다.
### 절차
1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다.
2. 생성자에서 이 부류의 객체들중 특정 객체를 정확히 찾는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다. 

## 9.6 [Magic Literal 바꾸기](https://refactoring.com/catalog/replaceMagicLiteral.html)
- 적절한 이름의 상수로 바꾸로 바꾸기
  - BadCase : `const ONE = 1` 
    - 의미전달에 변화가 없다.
    - 값이 달라질 가능성도 없다.
  - BadCase : 함수 하나에서만 쓰이고, 이해하기에 무리가 없다면 그냥 사용하자.
- 함수호출로 바꾸기
  ```java
  // ASIS
  if (a == "M") { ...}
  // 일반적으로 상수만 분리한다면
  if (a == MALE) {...}
  // TOBE, 명확한 의미전달 가능.
  if (isMale(a)) {...}
  boolean isMale(a) {
    return a == "M";    // 함수 내에서는 이해하기 명확하므로 그냥 사용해도 좋다.
  }
  ```
> String의 Magic Literal은 성능문제로 사용되었던것같은데, 찾아보자.