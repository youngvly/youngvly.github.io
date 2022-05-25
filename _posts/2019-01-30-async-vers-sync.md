---
title: "ASync is not the best way in always"
date: 2019-01-30 23:43:00 -0400
categories: JAVA
---

post요청과 delete요청을 해야하는 api가 있다.
한번의 요청을 보내는 기능도 있지만, 여러번의 post와 delete를 요청하는 기능을 구현할때, 
동기요청과 비동기요청 두가지를 고민중에 있다.

# Sync

restTemplate의 postForEntity를 차례로 호출하여 처리한다. (에러처리는 생략)
경우에따라 필요할 수 있는 순서대로 post요청이 가능하고 비교적 안정적이다.

```java
    someList.stream.foreach(restTemplate.postForEntity(url,request,String.class));
```

# ASync
구현 방법
1. AsyncRestTemplate 이용
    : Spring 4.0부터 추가되었으나 현재에는 Deprecated. WebClient가 대체하고있다. 
    Spring 5.0이 나와있는 시점에서 Deprecated된 api를 굳이 사용할 필요는 없다.
2. WebClient 이용
    : Spring 5.0 부터 나온 restTemplate과 AsyncRestTemplate의 역할을 모두 할 수 있는 확장된 api.
    - 현재 restTemplate을 이용하던 코드에서 많은 코드변경이 필요
        코드변경비용과 이점의 차이를 명확히 해야할필요가 있다. 
3. parallelStream 이용
    : 기존 stream()에서 병렬연산을 도와주는것으로 간단하게 병렬처리를 구현할 수 있다.


## 비교

- ASync가 Sync보다 수행시간이 단축되는것은 사실이다.
- ASync를 구현함에 있어서 thread pool 관리와 자원의 deadlock방지 등을 고려하여 코드를 구성해야하므로 동기처리보다 레이어복잡도가 훨씬 증가한다.
- ASync코드는 디버깅이 어려워진다. 
- ASync를 구성하기위한 코드를 작성하다가 Sync만한 효율도 못내는 경우도 있다.
- ASync가 진짜로 효능을 보이는 것은 수행하는 프로세스의 수행시간이 길 경우이다. 간단한 처리라면 위험을 감수하면서 비동기처리를 할 필요는 없다.

## 결론

- 비동기의 위험을 감수할만큼 현재 post와 delete요청을 처리하는데에 비동기처리가 필요한가?
- post,등록을 요청할때, 사용자가 입력한 순서대로 등록이 되는것이 일반적이다. 단지 반복작업의 속도를 줄이기 위하여 무작위로 등록이 되는것은 사용자가 원치않는 결과를 초래할수있다.
- delete 요청을 할때, 요청한 순서대로 delete가 되지 않을경우 문제가 발생할 가능성이 있다. 무작위로 delete되는것 또한 위험이 따른다.
- 일반적으로 get요청을 해서 빠르게 화면에 보여주어야 하는것이라면, 비동기가 필요할 수 있으나, post나 delete 와 같은 요청은 비동기보다는 동기처리가 정확한 일처리를 할 수 있다. 


## [추가] ParallelStream 의 문제


```java
    someList.parallelStream.map(() -> restTemplate.postForEntity(url,request,String.class)).collect(Collections.asList());
```

### 문제
- Default 로 ForkJoinPool.commonPool 를 사용하며 이는 Runtime.getRuntime().availableProcessors() 에서 반환하는 프로세서의 개수 -1개 만큼 쓰레드를 사용한다.
    (CPU 코어 개수만큼 반환한다.)
- 한 프로세서당 하나의 쓰레드를 사용한다.
- 이는 nested parallel stream 혹은 다수의 parallel stream 의 쓰레드들이 모두 하나의 풀을 공유한다고 할수있다.
- 최대로 사용했을때 모든 프로세서를 할당받지는 못한다.

### Thread 개수 해결방안 
  1.  system property 수정
```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "20")
```
    위 방법은 실행되는 모든 프로세스에 영향을 줄 수 있기때문에 사용하지 않는것이 좋다.

  2. Fork join pool을 새로 만드는 방법.
```java
ForkJoinPool myPool = new ForkJoinPool(5);
myPool.submit(() -> {
  IntStream.range(0, 10).parallel().forEach(index -> {
    System.out.println("Starting " + Thread.currentThread().getName() + ", index=" + index + ", " + new Date());
    try {
      Thread.sleep(5000);
    } catch (InterruptedException e) {
    }
  });
}).get(); 
```
위 방법으로 pool 크기를 정의하여 사용할 수 있다.
하지만 일반적으로는 common ForkJoinPool 을 사용하므로 해당 풀을 사용하는 다른 쓰레드에 영향을 줄 수 있다.
    
 + ForkJoinPool
  <img src="https://javatechnocampus.files.wordpress.com/2015/10/untitled.jpg">
  기본적으로 divide & Conqer 을 사용하며, 각 쓰레드가 개별 큐를 가지고 쓰레드가 비어있을때 fork, 일이 끝나면 join을 하는 방식이다.
  CPU가 최적의 성능을 발휘할 수 있다.

1. ForkJoinPool의 특성상 나누어지는 job은 균등하게 처리가 되어야 한다.
2. 병렬처리되는 작업들이 서로 독립적이지 않다면 위험하다.(내부적으로 자원을 공유하기때문.)
3. parallel stream 내부에서 다시 parallel stream 사용할 경우 synchronized 키워드는 deadlock 을 발생시킬 수 있다.
4. 특정 Container 내부에서 사용하는 경우에는 parallel은 신중하게 사용해야 하며, Container가 default pool을 어떻게 처리하는지 정확하게 모르는 경우에는 Default pool은 절대 사용하지 마라.
5. Java EE Container에서는 Stream의 parallel을 사용하지 마라.
   
### ... Spring과 같은 환경에서는 parallelStream을 사용해선 안된다.




[1] parallel stream : https://stackoverflow.com/questions/21163108/custom-thread-pool-in-java-8-parallel-stream
[2] Java8 Parallel Stream : https://www.popit.kr/java8-stream%EC%9D%98-parallel-%EC%B2%98%EB%A6%AC/
[3] http://itblues.pl/2016/10/31/asynchronous-request-processing-in-spring-mvc/
