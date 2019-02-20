---
title: "SpringBoot Cache & compression 설정"
date: 2019-02-19 19:00:00 -0400
categories: Spring-boot-study
---

### spring.resources.chain.cache
: Whether to enable caching in the Resource chain.
defualt = true

### spring.resources.cache.period
: Cache period for the resources served by the resource handler. If a duration suffix is not specified, seconds will be used.
max-age = value
참고 : https://stackoverflow.com/questions/1046966/whats-the-difference-between-cache-control-max-age-0-and-no-cache

### spring.resources.chain.enabled=true
: Enable the Spring Resource Handling chain. Disabled by default unless at least one strategy has been enabled.

### server.compression.enabled
: Whether response compression is enabled.
default = false

### server.compression.min-response-size
: 최소 압축 대상의 크기설정
default : 2KB (2048)

### server.compression.mime-types
: compression 할 대상
default : "text\/html",
        "text\/xml",
        "text\/plain",
        "text\/css",
        "text\/javascript",
        "application\/javascript",
        "application\/json",
        "application\/xml"



[문제]
properties에 아래와같이 설정하면 
        #spring.resources.cache.cachecontrol.max-age=3600
        #spring.resources.chain.enabled=true
favicon.ico , data? 만 캐싱되고
js파일들은 캐싱되지 않는 문제 발생.

[해결]
WebMvcConfigurer에서 설정.
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.
			.setCachePeriod(3600)
			.resourceChain(true);
	}
}
```