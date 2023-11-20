---
title: "TDD 1장 Money Example"
date: 2023-01-02 19:10:00 +0400
categories: test, study
tags: test, tdd
---

# 화폐예제
## 1. 다중 통화를 지원하는 Money 객체
1. 어떤 금액을(주가를) 어떤 수에(주식 수에)곱한 금액을 결과로 얻을 수 있어야 한다.
   1. 테스트 작성 (컴파일에러)
   2. 컴파일 에러 해결 : 클래스 작성, stub method/생성자 작성, 클래스 내 필드 작성\
   3. 테스트 실패 확인
   4. 테스트가 성공할 최소한의 변화 구현.
      1. 가짜로 구현하기 : 테스트를 통과시킬 상수를 반환하게 만들어 추후에 구현한다. (구현이 복잡할 경우 유용)
      2. 명백한 구현 사용하기: 실제 구현.
   5. 테스트 성공 확인
   6. 코드 중복 제거
        - 중복을 제거하면, 의존성이 제거된다. 다음 테스트로 진행하기 전에 중복을 제거하여, 한가지의 코드 수정을 통해 다음 테스트도 통과되게 만들 가능성을 최대화시키자.
```java
    @Test
    public void testMultiply(){
        Dollar five = new Dollar(5);
        five.times(2);
        assertThat(five.amount).isEqualTo(10);
    }
```
## 2. 불변객체 만들기 : 타락한 객체
1. 어떤 금액을(주가를) 어떤 수에(주식 수에) **여러번** 곱한 금액을 결과로 얻을 수 있어야 한다.
   1. 위 테스트의 오퍼레이션은(메서드) 부작용이 있으므로, 부작용이 없는 객체를 생성하자.
   2. 테스트 작성 (컴파일에러)
   3. ... 반복
```java
    @Test
    public void testMultiply(){
        Dollar five = new Dollar(5);
        assertThat(five.times(2).amount).isEqualTo(10);
        assertThat(five.times(3).amount).isEqualTo(15);
    }
```
## 3. VO의 오퍼레이션 구현하기 : 모두를 위한 평등
1. 값이 변화하지 않는 VO값객체는 equals를 구현해야 한다.
```java
    @Test
    public void testEquality(){
        assertThat(new Dollar(5)).isEqualTo(new Dollar(5));
        assertThat(new Dollar(5)).isNotEqualTo(new Dollar(17));
    }
```
## 4. private 필드 만들기 : 프라이버시
1. 클래스의 필드를 private로 변경해야한다.
   - `2. 불변객체 만들기` 에서의 테스트를 VO 특성을 활용하여 테스트한다.
   - equals 가 제대로 동작하지않으면, times() 메서드도 실패햘 수 있다.
```java
     @Test
    public void testMultiply(){
        Dollar five = new Dollar(5);
        assertThat(five.times(2)).isEqualTo(new Dollar(10));
        assertThat(five.times(3)).isEqualTo(new Dollar(15));
    }
```
## 5. Dollar대신 Franc
1. Dollar와 동일한 기능을 수행하는 Franc 클래스를 만들기위해 클래스를 복사한다.
```java
     @Test
    public void testMultiply(){
        Franc five = new Franc(5);
        assertThat(five.times(2)).isEqualTo(new Franc(10));
        assertThat(five.times(3)).isEqualTo(new Franc(15));
    }
```
## 6. 계층화 및 공통 메서드 분리
1. Money 클래스를 상위 계층으로 분리하자.
2. 공통 메서드 분리를 위해 두 클래스의 메서드를 일치시키자.
```java
    public class Money {
        private int amount;
    }
    public class Dollar extends Money {
        public boolean equals(Object object){...}
    }
    public class Franc extends Money {
        public boolean equals(Object object){...}
    }
```
## 7. 서로 다른걸 비교할 수 없다.
1. Dollar와 Franc 의 equals는 파라미터 클래스와 본인 클래스가 동일한지 확인해야한다.
```java
    @Test
    public void testEquality(){
        assertThat(new Dollar(5)).isEqualTo(new Dollar(5));
        assertThat(new Dollar(5)).isNotEqualTo(new Dollar(17));
        assertThat(new Franc(5)).isEqualTo(new Franc(5));
        assertThat(new Franc(5)).isNotEqualTo(new Franc(17));
        assertThat(new Dollar(5)).isNotEqualTo(new Franc(5));
    }
```
## 8. 객체 만들기
1. Dollar, Franc 같은 Money 하위 계층에 대한 참조가 적어진다면, 하위 클래스를 제거하기에 가까워졌다 할 수 있다.
   1. 팩토리 메서드를 도입하여 테스트 코드에서 콘크리트 하위 클래스의 존재 사실을 분리해 냈다.
```java
     @Test
    public void testMultiply(){
        Money five = Money.dollar(5);
        assertThat(five.times(2)).isEqualTo(new Dollar(10));
        assertThat(five.times(3)).isEqualTo(new Dollar(15));
    }
``` 

## 9. 통화 구현하고, 공통 생성자 상위로 올리기
- 공통 메서드인  구현을 일치시키고 상위 클래스인 Money로 올린다. 
- 작업중 발견되는 작은(간단한)수정들은 바로 수정해도 좋다. 적당한 수정이란 없는것, 본인이 하면서 올바른 정도를 찾아나가면 된다. 
```java
    public class Money {
        private int amount;
        private String currency;
        Money (int amount, String currency){
            this.amount = amount;
            this.currency = currency;
        }
        public static Dollar dollar(int amount){
            return new Dollar(amount,  "USD");
        }

    }
    public class Dollar extends Money {
        // 생성자는 factory method를 통해서만 사용할수있다
        Dollar(int amount, String currency){
            super(amount, currency);
        }
        public times(int time){
            return Money.dollar(this.amount * time);
        }
    }
```
## 10. 공통 메서드 올리기
- 메서드 구현을 일치시키기위해 호출메서드를 인라인시키고, 상수를 변수로 수정함.
- 디버깅을 위해 toString 을 테스트없이 작성할 수 있다. : 서비스 영향도가 작은 부분이라 예외라고 말함.
- Franc 대신 Money를 반환하게 하여 메서드를 상위로 올렸을때, 테스트가 깨짐을 확인하고, equals를 수정하여 돌아가게 수정.
  
```java
    public class Money {
        private int amount;
        private String currency;
        Money (int amount, String currency){
            this.amount = amount;
            this.currency = currency;
        }
        public Money times(int time){
            return new Money(this.amount * time, this.currency);
        }
        public boolean equals(Object object){
            Money money = (Money)object;
            return this.amount == money.amount && this.currency == money.currency;
        }
    }

    @Test
    void equalsTest(){
        assertThat(new Money(10,"CHF")).isEqualTo(new Franc(10,"CHF"));
        assertThat(new Money(10,"USD")).isEqualTo(new Dollar(10,"USD"));
    }
```

## 11. 하위 클래스 삭제
- 상위클래스로 메서드가 다 올라오니, 하위메서드에는 생성자만 남은상태이므로, 하위 클래스를 삭제한다.
- 상위 클래스로 깨지는 테스트를 대체하고, 불필요한 혹은 중복되는 테스트를 삭제한다.

## 12. 다른 통화의 합 기능추가
- 큰 테스트를 (5USD + 10CHF = 10USD) 작은 테스트로 바꾸어 ($5 + $5 = $10) 발전을 나타낼수있도록 한다.
- 계산에 대한 가능한 메타포를 신중히 생각하여 합의 결과를 나타내는 새로운 객체(지갑역할)과 한가지 통화로 환전해주는 객체를 분리한다.
    1. 나중에 다중 통화의 합은 어떻게 표현할 것인가.
    2. 달러와 같은 한가지 통화로 (참조통화) 통일시켜 계산하면 나중에 환율 표현이 어렵다.
    3. 여러 통화를 계산할 수 있는 새로운 객체가 필요하다. => Expression
```java
    @Test
    void testAddition(){
        Money five = Money.dollar(5);

        Expression sum = five.plus(five);       // 테스트하려고하는 핵심 부분이다. 핵심이 되는 객체가 다른 부분에 대해서 가능한 모르도록하면, 가능한 오래 유연할 수 있고, 테스트하기 쉬워질 수 있다. 책에서는 검증문부터 구현문~핵심부까지 역방향으로 테스트를 작성했다.
        Money reduced = new Bank().reduce(sum, "USD");
        
        assertThat(reduced).isEqualTo(Money.dollar(10));
    }

    class Money implements Expressions{
        public Expressions plus(Money addend){
            return new Money(amount + addend.amount, currency);
        }
    }
    class Bank {
        public Money reduce(Expression source, String toCurrency){
            return Money.dollar(10)
        }
    }
```
## 13. 구현하기
- 중복코드가 사라지기 전까지는 테스트가 돌아가도 완전한 것으로 치지 않는다.
- 미완성 테스트 안에 여러가지 구현이 요구될 경우 순방향으로 작업해보자.
    1. Money.plus 는 Sum 객체를 반환해야한다.
    2. Bank.reduce(sum,..) 은 Money 객체를 반환해야한다.
- 한곳에 클래스 캐스팅을 이용해 코드를 구현했다가, 테스트가 돌아가는것을 확인한 뒤 다형성으로 적당한 자리로 옮겼다.

1. Money.plus 는 Sum 객체를 반환해야한다.
```java
    @Test
    void testPlusReturnSum(){
        Money five = Money.dollar(5);
        Sum sum = five.plus(five);

        assertThat(sum.augend).isEqualTo(five);
        assertThat(sum.addend).isEqualTo(five);
    }

    class Sum implements Expression {
        Sum (Money augend, Money addend){
            this.augend = augend;
            this.addend= addend;
        }
    }
```

2. Bank.reduce(sum,..) 은 값이 더해진 Money 객체를 반환해야한다.
```java
    @Test
    void testBank(){
        Sum sum = new Sum(Money.dollar(5), Money(5));
        Money reduced = new Bank().reduce(sum, "USD");
    
        assertThat(reduced).isEqualTo(Money.dollar(10));
    }

    class Bank {
        // 첫번째 바로 구현시, 클래스 검사 및 캐스팅이 있었었으나, exp 인터페이스로 구현을 옮겼다.
        Money reduce(Expression exp, String currency){
            return exp.reduce(currency)
        }
    }

    class Sum implements Expression {
        Money reduce(String currency){
            return new Money(addend.amount + augend.amount, currency)
        }
    }

    interface Expression {
        Money reduce(String currency)
    }
```

## Bank에서 환전하기
- 환율에 대한 정보가 Money가 아는것은 어색하므로, 환전을 담당하는 Bank객체가 필요하다.
- reduce 할때 환율정보가 필요하므로, Bank객체를 받도록 한다.
```java
    @Test
    void testReduceWithDiffrentCurrency(){
        Bank bank = new Bank();
        bank.addRate("CHF", "USD", 2);
        Money money = bank.reduce(Money.franc(2),"USD");
        
        assertThat(money).isEqualTo(Money.dollar(1));
    }

    @Test
    void testReduceWithSameCurrency(){
        Bank bank = new Bank();
        Money money = bank.reduce(Money.dollar(2),"USD");
        
        assertThat(money).isEqualTo(Money.dollar(2));
    }

    class Money implements Expression {
        public Money reduce(Bank bank, String toCurrency){
            return new Money(amount / bank.rate(this.currency, toCurrency), toCurrency);
        }
    }

    class Sum implements Expression {
        Money reduce(Bank bank, String currency){
            return new Money(addend.amount + augend.amount, currency)
        }
    }

    interface Expression {
        Money reduce(Bank bank, String currency)
    }

    class Bank {
        // 책에서는 Array.equals가 원소에 대한 동치성검사를 하지않는것을 확인하고, Pair 객체의 equals, hashCode를 구현했다.
        private Hashtable<Pair,Integer> rates = new HashTables<Pair,Integer>();

        int rate(String from, String to){
            if (from.equals(to))  return 1;
            rates.get(new Pair(from,to));
        }
        void addRate(String from, String to, int rate){...}
    }
```

## 다른 통화로 환전하기
- 애초 구현하고자했던 `5USD + 10CHF = 10USD` 원하는 테스트를 작성한 뒤, 구현한다.
- 추상화를 사용하여 부모객체에서 기능을 제공하도록 한다. (Money -> Expression 으로 가능하면 치환)
```java
    @Test
    void testMixedAddition(){
        Expression fiveBucks = Money.dollar(5);
        Expression tenFranc = Money.franc(10);
        Bank bank = new Bank().addRate("CHF", "USD", 2);
        
        Sum sum = fiveBucks.plus(tenFranc);
        Money result = bank.reduce(sum, "USD");

        assertThat(result).isEqualTo(Money.dollar(10));
    }

    interface Expression {
        Expression plus (Expression addend) {..}
        Expression reduce(Bank bank, String currency)
    }

    class Sum implements Expression {
        Expression reduce(Bank bank, String to){
            return new Money(addend.reduce(bank, to).amount + augend.reduce(bank, to).amount, to)
        }
        
        Expression plus(Expression addend){
            return null; //TODO
        }
    }

    class Money implements Expression {
        Expression plus (Expression addend) {..}
        Expression times (int time){..}
        Expression reduce(Bank bank, String currency){..}
    }
```

## 16. 추상화 완성하기
1. 위에서 미완성되었던 `Sum.plus` 메서드를 구현한다.
```java
    @Test
    void testSumPlusMoney(){
        Expression fiveBucks = Money.dollar(5);
        Expression tenFranc = Money.franc(10);
        Bank bank = new Bank().addRate("CHF", "USD", 2);
        
        Sum sum = fiveBucks.plus(tenFranc).plus(fiveBucks);
        Money result = bank.reduce(sum, "USD");

        assertThat(result).isEqualTo(Money.dollar(15));
    }

     class Sum implements Expression {
        Expression plus(Expression addend){
            return new Sum(this, addend);
        }
    }
```
2. `times` 메서드도 추상화시킨다
```java
    @Test
    void testSumTimes(){
        Expression fiveBucks = Money.dollar(5);
        Expression tenFranc = Money.franc(10);
        Bank bank = new Bank().addRate("CHF", "USD", 2);
        
        Sum sum = fiveBucks.plus(tenFranc).times(2);
        Money result = bank.reduce(sum, "USD");

        assertThat(result).isEqualTo(Money.dollar(20));
    }

    interface Expression {
        Expression plus (Expression addend)
        Expression times (int time)
        Expression reduce(Bank bank, String currency)
    }
    
    class Sum implements Expression {
        Expression plus(Expression addend){
            return new Sum(this, addend);
        }

        Expression times(int time) {
            return new Sum(augend * times(time) , addend * times(time));
        }
    }

    public class Money implements Expression {
        Expression times(int time){
            return new Money(this.amount * time, this.currency);
        }
    }
```
3. Money + Money = Money 객체 구현시도
    - 현재 구조로는 Sum 객체를 반환중이다. 객체의 행위에 대한 내용이 아니라, 자세한 구현 내용에 의한 테스트이므로 값어치가 없다.
    - 책에서는 위 요구사항은 버린다. 
- TDD로 구현할땐, 테스트코드와 모델코드의 줄 수가 비슷한 상태로 끝난다.
- TDD가 경제적이기 위해서는 코드줄수가 두배가 되거나 동일한 기능을 구현하되, 절반의 줄수로 해야할것이다.
 > 뭔말
- 디버깅,작업,코드설명시간을 포함한 일반적인 자신의 방법과, TDD가 어떻게 다른지 직접 축정해보자.

# 17. 회고
### 다음에 할 일은 무엇인가
- = 어떤 테스트가 추가로 필요할까?
- 위 money 에제에서는 `Sum.plus` `Money.plus` 두 메서드가 구현이 동일하기때문에, 공통코드를 상위로 뽑아낼 방법을 고안해볼 수 있다.
- 실패해야하는 테스트가 성공한다면, 이유를 찾아내야한다. 반대도 마찬가지
- 할일 목록이 빌때 설계를 검토하기 좋은 시기이다.

### 메타포
- Expression 인터페이스를 분리하여 추상화함으로써 얻은 이득이 크다. 중복코드 없음, 구현의 단순화.

### Junit 사용도
- 위 예제 작성하면서 테스트를 1분간격으로 실행하여 125번 실행했다. > 그만큼 테스트를 많이 돌려봤다 정도

### 코드 메트릭스
- 실제 코드와 테스트 코드의 줄수, 회기 복잡도 등의 수치가 비슷하다.

### 프로세스
- TDD의 주기
    1. 작은 테스트 추가
    2. 모든 테스트 실행, 실패하는것 확인
    3. 코드 구현
    4. 모든 테스트 실행, 성공하는것을 확인
    5. 중복을 제거하기 위해 리팩토링
       1. 리팩토링당 수정해야하는 횟수는 적거나, 매우크거나 종형곡선이 나타난다. (메서드나 클래스 정의를 바꾸는 것)

### 테스트의 질
- 기능테스트가 성능테스트/스트레스 테스트/사용성 테스트 를 대신하진 않는다.
- TDD는 명령문 커버리지 100%를 추구한다. 
  - Money 예제는 toString 이외에는 100% 이다.
- 결함 삽입 방법  : 코드의 의미를 바꾼 후에 실패하는지 확인하는 방법
  - Jester 내용을 변경해도 바뀌지않는 곳을 찾아주는 툴.
  - Money 예제에서 hashCode() { return 0 } 인부분을 찾아냈지만, 실제 코드에서 구현필요성이 없는 부분이므로 괜찮다.
- 테스트 커버리지를 높이는 방법
  - 테스트 수를 늘린다.
  - ** 리팩토링으로 프로그램 로직을 단순화한다. **

### 최종 검토
- 테스트를 확실하게 돌아가게 하는 방법
  - 가짜로 구현하기
  - 삼각 측량법
  - 명백하게 구현하기
- 설계를 주도하기 위한 방법으로 테스트코드와 실제 코드사이의 중복을 제거하기
- 길이 미끄러우면 속도를 줄이고, 상황이 좋으면 속도를 높이는 식으로 테스트 사이의 간격을 조절할수있는 능력