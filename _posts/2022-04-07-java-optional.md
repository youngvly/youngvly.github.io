---
layout: post
title: "Java Optional 제대로 사용하기"
date: 2022-04-07 19:10:00 +0400
categories: JAVA
tags: java
---
# Optional을 제대로 써야하는 이유

[26 Reasons Why Using Optional Correctly Is Not Optional - DZone Java](https://dzone.com/articles/using-optional-correctly-is-not-optional)

> Optional은 메서드의 리턴타입이 ‘리턴값이 없다’ 는 것을 명확하게 보여주기위한 제한된 메커니즘을 제공하기위해 의도된것이다. null을 반환하는것은 압도적으로 많은 에러를 유발한다.
> 
1. Optional변수를 null로 세팅하지말라.

```java
//NO
Optional<Car> emptyCar = null;
//OK
Optional<Car> emptyCar = Optional.empty();
```

2. Optional.get() 호출전까지 값이 있도록 하라.
아니면 isPresent() 체크를 같이 해라. 물론 이것도 좋은 방법은 아님.

```java
//OK
if (car.isPresent){
	Car car = carOptional.get();
}
```

3. 값이 없다면 미리 생성된 `orElse` 로 default 객체를 반환하라. (ifPresent() - get() 보다 추천)`
4. `orElse`는 불필요할 수도 있는 default객체가 필요하므로, `orElseGet(Supplier a)`으로 필요할때만 객체를 생성하라.

```java
//OK
carOptional.orElse(Car.empty())
//OK GOOD
carOptional.orElseGet(Car::new)
```

5. 값이 없으면 `NoSuchElementException`을 throw하는(JDK10+) `orElseThrow()` 를 사용하라.
    1. 이것도 기본 exception을 던지면 좋지만, 필요시 supplier로 exception을 생성하자.
6. null 반환이 필요하면 orElse(null)로 사용하자. (ifPresent 말고)
7. 값이 있을때만 수행할 무언가가 필요하면 `ifPresent(consumer)`를 사용하자. isPresent()-get()말고
8. 값이 있든 없든 무언가 수행해야한다면 `ifPresentElse()`를 사용하자. (JDK9+) 

```java
carOptional.ifPresentOrElse(
	log::error,
  () -> log.error("no!")
}
```

 10. 값이 없어도 Optional로 감싼 default 값이 필요할때 Optional.or() (JDK9+)

```java
public Optional<Car> safeCar(){
	return carOptional.or(() -> Optional.of(new Car()));
}
```

1. if분기로 코드를 어지럽히는 `isPresent()-get()` 보다 `orElsexxx()` 를 사용하자.
2. 간단한 값을 가져오는 로직을 위해 Optional을 남용하지말자.

```java
//No
return Optional.ofNullable(status).orElse("PENDING");
//OK
return status == null ? "PENDING" : status;
//OK2
return Objects.requireNonNullElseGet(status,() -> "PENDING");
```

1. Optional을 프로퍼티나 파라미터로 사용하지말자. Serializable하지 않고, 필드사용을 위해 생긴것이 아니다.
2. Optional을 생성자 파라미터로 사용하지말자. setter 포함.
Optional의 의도에 맞지않다.
getter에 Optional을 사용하는것도 over-use다.
3. Optional을 setter 파라미터로 사용하지말자.
4. Optional을 메서드 파라미터로 사용하지말자.
코드를 더 복잡하게만들고, Optional을 추가로 생성하게 만든다. 생성비용이 최소참조 비용보다 4배더 든다.
5. empty Collection이나 Array 반환을 위해 Optional을 사용하지말자.
Collections.emptyxxx() 가 있다.
6. Optional을 collection에 담지말자. 
설계오류이다. Optional의 비용이 크고, 기본 메서드를 잘 활용하자. 

```java
//No
Map<String,Optional<String>> no!;
//Yes
Map<String,String> yes=..
map.getOrDefault(key,”default");
```

1. Optional.of() 와 ofNullable을 실수로 사용하지말자. 
Optional.of(null)은 NPE를 발생시킨다.
Optional.ofNullable(null) 은 Optional.empty()와 같다.
2. primitive 타입이라면 Generic Optional<T> 보다 OptionalInt, Optionalxx 를 사용하자. 
boxing, unboxing은 성능문제를 야기할수있다.
3. Optional 내부값을 비교하기위해 get을 호출하지않고, Optional.equals로 비교할 수있다.

```java
Optional<String> big = Optional.of("Shoes");
Optional<String> small = Optional.of("shoes");
//NO
assertEquals(big.get(),small.get());
//YES
assertEquals(big,small);
```

1. map(), flatMap() 메서드를 잘 활용해서 값을 바꾸자.
2. filter() 메서드를 잘 활용해서 필터링하자.
3. stream 에서 Optional을 사용해야할때, Optional.stream() 을 사용하자(JDK 9+)
존재하는 값의 stream을 생성한다. Optional이 비어있으면 empty Stream

```java
//ASIS
list.stream()
	.map(this::someReturnsOptional)
	.filter(Optional.isPresent)
	.map(Optional::get)
	.collect(toList());
//TOBE
list.stream()
	.map(this::someReturnsOptional)
	.flatMap(Optional::stream)
	.collect(toList());
```

1. equality(==) , identity hash-based, synchronization을 사용하지말자.
Optional은 [value-based class](https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html)이다 

```java
// NEVER!!
Optional<Car> car = Optional.of(new Car());
synchronized(car) {
	...
}
```

1. 값이 있는지 boolean을 반환해야한다면 Optional.isEmpty()를 사용하자 (JDK11+)

> **The young man knows the rules, but the old man knows the exceptions.**
>