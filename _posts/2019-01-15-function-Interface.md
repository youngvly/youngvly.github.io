---
title: "Function Interface"
date: 2019-01-14 23:00:00 -0400
categories: java-study
---

# 함수형 인터페이스

- 디폴트 메서드 :  인터페이스의 메서드를 구현하지않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드
- Functional Interface : 디폴트 메서드가 많더라도 추상메서드가 오직 하나인 인터페이스.
- Function descriptor :  함수형 인터페이스의 추상메서드 시그니처.
- 
## 1. Predicate<T>

T 타입의 객체를 받아서 boolean 을 반환하는 boolean test() 추상메서드가 있다.

```java
Predicate<String> nonEmptyStringPredicate = (String s)-> !s.isEmpty();
nonEmptyStringPredicate.test("hi") //--> true
```
- Predicate<int>은 불가능 -> Predicate<Integer> 으로 박싱해야한다.
- 기본형을 참조형으로 박싱하면 박싱한 값은 힙에 저장된다. -> 메모리를 더 소비하며 기본형을 가져올때에도 탐색비용이 든다.
- 오토박싱을 방지하기위한 interface : IntPredicate , LongPredicate , DoublePredicate ..
```java
Predicate<Integer> predicate = (Integer i) -> i> 10;
IntPredicate intpredicate = (int i)-> i > 10 ; 
```

## 2. Consumer<T>

T 타입의 객체를 받아서 void를 반환하는 void accept() 추상메서드가 있다.
```java
Consumer<Integer> printInteger = (Integer i)-> System.out.println(i);
```

## 3. Function<T,R>

T타입의 객체를 받아서 R타입을 반환하는 R apply() 추상메서드가 있다.
```java
Function<String,Integer> stringLength = (String s)->s.length();
```
- Function 인스턴스를 반환하는 andThen, compose 가 있다.

# 메서드 레퍼런스

ClassName::instanceMethod
기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

```java
Function<String, Integer> integerParser1 = (String s) -> Integer.parseInt(s);
Function<String, Integer> integerParser2 = Integer::parseInt    //메서드 레퍼런스 
```

  