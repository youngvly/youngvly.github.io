---
title: "WebFlux Thread DeadLock"
date: 2020-12-29 21:57:00 -0400
categories: SpringBoot
tags: rxJava, springBoot, netty
---

# WebFlux Thread lock

사용중인 코드

```java
Webclient()
...
.retrieve()
.bodyToMono(SomeClass.class)
.block()
```

쓰레드 덤프

```bash
"http-nio-8000-exec-16" Id=0x54 WAITING on java.util.concurrent.CountDownLatch$Sync@4e78b2fd
    at sun.misc.Unsafe.park(Native Method)
    -  waiting on java.util.concurrent.CountDownLatch$Sync@4e78b2fd
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:997)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
    at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:231)
    at reactor.core.publisher.BlockingSingleSubscriber.blockingGet(BlockingSingleSubscriber.java:81)
    at reactor.core.publisher.Mono.block(Mono.java:1183)
```

관련 이슈

1. [Mono#block() and Mono#blockOptional() may occasionally produce deadlock. · Issue #1651 · reactor/reactor-core](https://github.com/reactor/reactor-core/issues/1651)

2. [Sometimes deadlock is occurred due to blocking DNS resolver · Issue #710 · reactor/reactor-netty](https://github.com/reactor/reactor-netty/issues/710)


### 해결 방안

DNS 캐싱관련 발생 이슈로, netty의 async DNSResolver을 사용하면 해결가능하다.

AsyncDNSResolver는 netty 4.1.56 부터 제공된다.

Netty DNS를 디폴트로 사용하는 reactor-netty 1.0 버전을 사용하면 추가 설정없이 해결 가능하다.


### 참고

Deadlock이 발생할수있는 상황 및 Nonblocking thread에서 block이 일어났을때 감지 및 방지해줄수있는 BlockHound 

[Reactor Debugging Experience](https://spring.io/blog/2019/03/28/reactor-debugging-experience#blockhound-a-new-kid-on-the-block)