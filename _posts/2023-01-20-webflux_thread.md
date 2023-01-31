---
title: "Webflux thread"
date: 2023-01-20 19:10:00 +0400
categories: Webflux, Spring
tags: webflux, spring
---

mono/flux 에서 block시 데드락이 걸리는현상을 개선중, controller로 mono/flux가 직접 반환될때, 그럼 api요청을 받은 스레드와 webclient에서 생성한 스레드가 둘다 생길텐데, api 요청을 받은 스레드는 먼저 종료되는것인가?

# return Mono.just
```java
    @GetMapping("/v1/delay-test")
	public Mono<String> delay(@RequestParam int delaySeconds) {
		log.debug("out of mono");
		return Mono.just("ok")
			.map(t -> {
				log.debug("in mono");
				return t;
			})
			.delayElement(Duration.ofSeconds(delaySeconds));
	}
```
api 응답이 mono이고, 일반 mono를 생성하여 그대로 리턴할때, 같은 스레드에서 동작된다.
```sh
[     parallel-1] Created new WebSession.
[     parallel-1] out of mono
[     parallel-1] in mono
```

# return Webclient.bodyToMono()
```java
    @GetMapping("/v1/delay-test")
	public Mono<String> delay(@RequestParam int delaySeconds) {
		Thread thread = Thread.currentThread();
		log.debug("out of mono, thread : {} , {}", thread.getName(), thread.isAlive());
		return WebClientUtil.get("http://test.com", 5)
			.get()
			.uri("/test/api")
			.retrieve()
			.bodyToMono(String.class)
            .delayElement(Duration.ofSeconds(10))
			.map(t -> {
				log.debug("in mono: {} , {}", thread.getName(), thread.isAlive());
				Thread thread2 = Thread.currentThread();
				log.debug("in mono: {} , {}", thread2.getName(), thread2.isAlive());
				return t;
			});
	}
```
webclient 에서 새로운 스레드를 할당받는데, 그렇다고 기존 세션스레드가 사라지는것은 아니다.
ㄴ 그럼 `Accept: text/event-stream` , `Accept: application/stream+json` 으로 요청시 먼저오는 응답은 먼저받을수있다는 장점 빼고는?
```sh
[     parallel-1] Created new WebSession.
[     parallel-1] out of mono, thread : parallel-1 , true
[ctor-http-nio-4] [842502c8] Connecting to xxx.
[ctor-http-nio-4] in mono: parallel-1 , true
[ctor-http-nio-4] in mono: reactor-http-nio-4 , true
[ctor-http-nio-4]  Releasing channel
```

# subscribe
```java
    @GetMapping("/v1/delay-test")
	public Mono<String> delay(@RequestParam int delaySeconds) {
		Thread thread = Thread.currentThread();
		log.debug("out of mono, thread : {} , {}", thread.getName(), thread.isAlive());
		WebClientUtil.get("http://test.com", 5)
			.get()
			.uri("/test/api")
			.retrieve()
			.bodyToMono(String.class)
			.delayElement(Duration.ofSeconds(10))
			.map(t -> {
				log.debug("in mono: {} , {}", thread.getName(), thread.isAlive());
				Thread thread2 = Thread.currentThread();
				log.debug("in mono: {} , {}", thread2.getName(), thread2.isAlive());
				return t;
			})
			.subscribe(s -> log.debug("end!!"));

		return Mono.just("result");
	}   
```
api는 먼저 종료되어 응답이 먼저 나가고, subscribe 는 이후에 따로 처리된다. 스레드는 종료까지 살아있다.
```sh
[     parallel-1] Created new WebSession.
[     parallel-1] out of mono, thread : parallel-1 , true
[ctor-http-nio-4] [4bd9880f] Created a new pooled channel,
[     parallel-1] [7ee538f9-1] Writing "result"
[ctor-http-nio-4] [4bd9880f] Releasing channel
... 10s 뒤
[     parallel-2] in mono: parallel-1 , true
[     parallel-2] in mono: parallel-2 , true
[     parallel-2] end!!
```