---
title: "RXJava Observable"
date: 2020-12-29 18:07:00 -0400
categories: rxJava
tags: rxJava, observable
---

# Table of Contents
1. [Observable](#Observable)
2. [ConcatMap](#concatMap)
3. [FlatMap](#FlatMap)
4. [publish](#publish)
5. [ConnectableObservable](#ConnectableObservable)
6. [refCount](#refCount)

# Observable

```java
import rx.Observable;

private final Random rnd = new Random();

private final Observable<Temperature> dataStream =
	Observable.range(0,Integer.MAX_VALUE)
		.concatMap(tick -> Observable.just(tick)
													.delay(rnd.nextInt(5000),MILLISECONDS)
													.map(tickValue -> this.probe()
							) // 만들어진 각 측정사이의 max 5초간의 센서값을 반환받을 수 있다.
		.publish()// ConnectableObservable<Temperature>
		.refCount(); // Observable<Temperature> -> 구독자가 없을때 센서를 탑색하지않도록 할 수 있다.

private Temperature probe() {
		return new Temperature(16 + rnd.nextGaussian() * 10);
}
```

## Observable :

구독가능한 대상 ,이벤트 발행객체

### concatMap

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/concatMap.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/concatMap.png)

```java
public final <R> Observable<R> concatMap(Func1<? super T, ? extends Observable<? extends R>> func) {
```

T 타입을 받아 Observable 로 반환하여, 결과 Observable에 순차적으로 병합된다.

### FlatMap

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/flatMap.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/flatMap.png)

```java
public final <R> Observable<R> flatMap(Func1<? super T, ? extends Observable<? extends R>> func)
```

T타입을 받아 Observable로 변환하나, 순차적 병합이 보장되지 않는다.

### publish

![https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/publishConnect.png](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/publishConnect.png)

```java
public final ConnectableObservable<T> publish() {
```

Observable → ConnectableObservable 로 변환해준다.

## ConnectableObservable

subscriber의 connect() 메서드가 호출되기 전까지, observer는 이벤트를 발행하지 않는다.

### refCount

![http://reactivex.io/documentation/operators/images/publishRefCount.c.png](http://reactivex.io/documentation/operators/images/publishRefCount.c.png)

```java
public Observable<T> refCount() {
```

ConnectableObservable이 Observable처럼 행동할수있게 반환한다.

Observable().publish().refCount() 으로 Observable이 subscriber의 구독이후부터 이벤트가 발행될수있도록 할 수 있다.