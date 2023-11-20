---
layout: post
title: "Refactoring 07 캡슐화"
date: 2022-08-08 23:59
categories: Refactoring study
tags: cleanCode, study
---
# 07 캡슐화
## 7.1 레코드 캡슐화하기
- 가변데이터 : 객체
- 불변데이터 : 레코드
    - 필드 이름을 노출하는 형태
    - 내가 원하는 이름을 쓰는 형태 (hashMap) : 사용하는 곳이 많을 수록 불분명함으로 문제가 발생.
- 중첩된 리스트나 해시맵을 직렬화시키는 경우에도, 캡슐화하면 포맷을 바꾸거나 추적하기 어려운 데이터 수정에 쉽다.
- 접근할때에는 객체의 접근자를 사용하자.
- 클래스에서 원본데이터를 반환하는 접근자와 원본 레코드를 지우자
   - deepClone
   - 내부객체를 수정하려고할때 예외를 던지는 proxy
   - 복제본을 만들고 이를 재귀적으로 동결(freeze) 해서 쓰기동작을 감지하는 방법.(??) (p.243)
## 7.2 컬렉션 캡슐화하기
- 컬렉션을 소유한 클래스를 통해 컬렉션에 접근하도록 하기 (add/remove)
- getter에서 복사본을 제공하기 (unmodifiable..)
- 중요한건 코으베이스에서 일관성을 주는것, 한가지만 적용해서 컬렉션 접근 함수의 동작 방식을 통일해야한다.
> 방식을 통일하는게 중요할듯
> lombok의 unmodifiableCollection getter 제공여부에 관한 [논의](https://github.com/rzwitserloot/lombok/issues/1504) , 
    - 제공하려면 전체 다 제공하거나 아니어야한다.
    - collection만의 문제가 아니라, 일반 객체도 deepClone되어야한다.
    - 일반객체도 모두 제공되기는 어렵다.
## 7.3 기본형을 객체로 바꾸기
- 단순 출력 이상의 기능이 필요해지면 데이터를 표현하는 전용 클래스를 정의하자.
- 경험 많은 개발자들은 여러 리팩터링중에서도 가장 유용한것으로 뽑는다.
```java
// ASIS
highPriorityCount = orders.map(order::getPriorityString).filter(ps -> ps == 'high' || ps == 'rush').length()
// TOBE
class Priority {
    private var static priorities = List.of("normal","high","rush")
    private int index;
    boolean higherThan(Priority p) {...}
}

highPriorityCount = orders.map(order::getPriority).filter(p -> p.higherThan(new Priority("normal")).length()
``` 
## 7.4 임시변수를 질의함수로 바꾸기
- 스냅샷 용도의 변수는 리팩토링 대상이 아니다.
- 변수가 여러차례 대입된다면 질의함수로 바꾸자.
```java
func discountFactor () {
    var factor = 0.98
    if (this.basePrice > 1000) factor -= 0.03
    return factor
}
```
## 7.5 클래스 추출하기
- 데이터나 메서드 일부를 제거했을때 논리적으로 문제가 없다면 분리하자.
- 서브클래스 생성시, 작은 일부 기능만을 위해 서브클래스를 만들거나 확장해야할 기능이 무엇이냐에 따라 서브클래스 만드는 방식이 달라진다면 클래스를 나누자.
- 
## 7.6 클래스 인라인하기
- 리팩터링 후 역할이 사라진 클래스는 인라인하자
- 리팩터링(클래스 추출) 전 기능을 다르게 배분하고싶을때 인라인하여 합쳐보자.
## 7.7 위임 숨기기
- 반대 : 중개자 제거하기 (7.8)
- 클라이언트가 위임객체의 존재를 알지 못하도록 하자. 위임객체의(department) 인터페이스가 바뀌면 사용하는 전체 코드가 변경되어야한다. 
```java
// ASIS
manager = person.department.manager;
// TOBE
manager = person.manager;
``` 
## 7.8 중개자 제거하기
- 반대 : 위임 숨기기
- person 클래스가 중개자의 역할만 하게될 수 있어, 위임객체를 직접 호출하는게 나을 수도 있다.
- 7.7과 7.8이 적당히 섞여도 좋다.
## 7.9 알고리즘 교체하기
- 더 간명한 방법을 찾아내면 수정하자.
- ex) 같은 기능을 제공하는  라이브러리 사용
- 메서드를 잘게 나누어야 알고리즘 교체가 쉬워진다.