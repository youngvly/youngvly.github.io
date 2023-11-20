---
layout: post
title: "Tips for webflux"
categories: SpringBoot summary
tags: rxJava, springBoot, netty
---

할때다 헷갈리는 webflux

### `Mono<Void>` 는 onNext, map 을 호출하지 않는다.
[see](https://github.com/reactor/reactor-core/issues/1467)

- 어차피 void 에서 map 될게없으니, Mono 내용 변환이 목적이라면 then으로 끊고가자.
- then : 이전 mono결과를 무시하고, 새로운 mono로 생성하여 이어간다.

### Compose와 Transform의 차이
- Compose는 한번의 subscribe가 호출된 후 적용된다.
- Transform은해당 mono에서 바로 적용된다.
- [see](https://stackoverflow.com/questions/47348706/compose-vs-transform-vs-as-vs-map-in-flux-and-mono/48203265#48203265)