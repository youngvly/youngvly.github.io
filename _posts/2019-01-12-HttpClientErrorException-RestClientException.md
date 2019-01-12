---
title: HttpClientErrorException and RestClientException"
date: 2019-01-12 22:37:00 -0400
categories: Spring-boot-study
---
# RestClientException

- RestTemplate이 발생시키는 client-side Http 에러.
- RuntimeException

## HttpStatusCodeException
### + HttpClientErrorException
- HttpStatusCode 4xx을 받았을때 발생시키는 에러. 

### + HttpServerErrorException
- HttpStatusCode 5xx을 받았을때 발생시키는 에러.


# RestTemplate Handling
1. try/catch block으로 custom error handling을 적용시킨다. -> 확장의 어려움.
2. restTemplate에 ResponseErrorHandler을 implements 한 errorHandler를 직접 적용시킨다.
```java
public class SomeClass {
    private RestTemplate restTemplate;
    public SomeClass(RestTemplateBuilder restTemplateBuilder) {
        restTemplate = restTemplateBuilder
                        .errorHandler(new someExceptionHandler())
                        .build();
    }
}
```
someExceptionHandler의 예시
```java
@Component
public class someExceptionHandler implements ResponseErrorHandler{
    @Override
    public boolean hasError(ClientHttpResponse httpResponse) throws IOException{
        return (httpResponse.getStatusCode().series() == CLIENT_ERROR ||
            httpResponse.getStatusCode.series() == SERVER_ERROR
        );
    }

    @Override
    public void handleError(ClientHttpResponse httpResponse) throws IOException {
        if (httpResponse.getStatusCode().series() == HttpStatus.Series.SERVER_ERROR){
            //handle SERVER_ERROR
        }else if (httpResponse.getStatusCode().series() == HttpStatus.Series.CLIENT_ERROR){
            //handle CLIENT_ERROR
            if (httpResponse.getStatusCode() == HttpStatus.NOT_FOUND){
                throw new NotFoundException();
            }
        }
    }
} 
```


[1] java.doc : https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/HttpClientErrorException.html
[2] Spring RestTemplate Error Handling : https://www.baeldung.com/spring-rest-template-error-handling 