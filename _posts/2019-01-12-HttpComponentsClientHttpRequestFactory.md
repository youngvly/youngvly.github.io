---
title: "HttpComponenetsClientHttpRequestFactory 란"
date: 2019-01-12 22:37:00 -0400
categories: SpringBoot
---
# HttpClient
- Http프로토콜통신을 할수있도록 도와주는 client side API

- HttpClient 4.x 부터는 Apache HttpComponents로 불린다.
- 모든 Http method를 지원한다.
- Https 프로토콜의 암호화기능을 지원한다.
- 멀티스레드 프로그램에서 사용될수있는 connection management.
- 쿠키사용 지원.
- ..

# RestTemplate
- RestTemplate는 HttpClient를 추상화하여 (HttpEntity의 json, xml 등) 제공한다. 
- 내부에서는 HttpComponents를 사용한다. 
- RestTemplate는 ClientHttpRequestFactory로부터 ClientHttpRequest를 가져와서 요청을 보낸다.
- ClientHttpRequest는 요청메세지를 만들어 HTTP프로토콜을 통해 서버와 통신한다.
  
<img src="http://img1.daumcdn.net/thumb/R960x0/?fname=http%3A%2F%2Fcfile26.uf.tistory.com%2Fimage%2F99300D335A9400A52C16C1" />

## ConnectionPool
- RestTemplate는 ConnectionPool을 사용하지 않는다. 
- 연결할때마다 로컬포트를 열고 tcp Connection을 맺는다.
- 이때 connection close 이후 사용된 소켓은 TIME_WAIT상태가 되는데, 소켓을 재사용하지 못한다.
- 요청량이 많아지면 이러한 소켓을 재사용하지 못하고 소켓이 오링이 나서 응답이 지연될 것이다.
- RestTemplate 에서 Connection Pool을 만들려면? HttpClient를 만들어주어야한다.
  
# ClientHttpRequestFactory 

- RestTemplate에 HttpClient를 적용시키기 위해선 ClientHttpRequestFactory가 필요하다.
- public RestTemplate(ClientHttpRequestFactory requestFactory);
- ClientHttpRequest : client-side Http 요청을 나타내는 클래스.

+ *implementing Class : AbstractClientHttpRequestFactoryWrapper, BufferingClientHttpRequestFactory, HttpComponentsAsyncClientHttpRequestFactory, HttpComponentsClientHttpRequestFactory, InterceptingClientHttpRequestFactory, MockMvcClientHttpRequestFactory, Netty4ClientHttpRequestFactory, OkHttp3ClientHttpRequestFactory, SimpleClientHttpRequestFactory*

## HttpComponentsClientHttpRequestFactory

- ClientHttpRequestFactory 를 implements하는 클래스.
- Request를 만들기 위해 Apache HttpComponents HttpClient를 사용하는 팩토리. 
- HttpClient를 instance로 가지고있으며, 지정하지않을경우 default.

```java
HttpClient httpClient = HttpClientBuilder.create()
    .setMaxConnTotal(100)
    .setMaxConnPerRoute(5)
    .build();

HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
factory.setReadTimeout(1000)        //읽기시간, ms
factory.setConnectTimeout(1000)     //연결시간 , ms

RestTemplate restTemplate = new RestTemplate(factory);
```
Connection Pool 제어 부분
- setMaxConnPerRoute : IP포트 1쌍에 대해 수행할 연결 수를 제한한다.
- setMaxConnTotal : 최대 오픈되는 커넥션 수를 제한한다. 

# HTTPatch
: Apache HttpComponents HttpClient 의 4.2버전부터 등장한 HttpMethod

1. SimpleClientHttpRequestFactory
    - RestTemplate 의 Default requestFactory
    - 표준 JDK를 활용한 ClientHttpRequestFactory
    - HttpClient의 4.2버전을 지원하지 않는다.
    - Patch request를 만들 수 없다.
  
2. HttpComponentsClientHttpRequestFactory
    - request를 만들기 위해 Apache HttpComponents HttpClient를 사용하는 ClientHttpRequestFactory.
    - Apache HttpComponents 4.3 이상의 버전이 요구된다.

**: RestTemplate에서 HttpMethod.PATCH를 사용하기 위해서는 HttpComponentsClientHttpRequestFactory를 적용시켜주어야한다.**

*참고*
[1] RestTemplate 정의 및 .. : http://sjh836.tistory.com/141
[2] HttpClient : https://hc.apache.org/httpcomponents-client-ga/ 
[3] RestTemplate : https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html 
[4] HttpPatch : http://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/methods/HttpPatch.html
[5] HttpComponentsClientHttpRequestFactory : https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/HttpComponentsClientHttpRequestFactory.html
[6] SimpleClientHttpRequestFactory : https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/client/SimpleClientHttpRequestFactory.html 
[7] Reason of changing requestFactory - Comment : https://jira.spring.io/browse/SPR-7985?focusedCommentId=80924&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-80924  