---
title: "SpringBoot와 webpack연동시 발생한 문제"
date: 2019-02-19 17:00:00 -0400
categories: Spring-boot-study
---

# Spring Boot 와 webpack의 연동


### 방법1
*dev-server를 두어 3000포트에서 8080포트로 들어오는 것들을 프록시해준다.*
live server가 되어 js의 즉각적인 수정이 가능하다.
develop 환경에서 유용.

### 방법2
*build. 기존 static 파일들을 번들하여 /src/main/resources/static 폴더에 넣어줄수있다.*


## 발견된 문제와 한계점
1. [한계] html Webpack plugin을 사용하지 못한다 
    -> chunk를 분리하여 html파일에 <script src="filename"/> 해주어야하는데 사용되는 플러그인, html파일에 해당라인을 추가하여 만들어준다.
    -> spring boot의 컨트롤러에서는 return "templateName" 을 하므로 templateName.html파일이 없으면 에러가 난다.
    -> 동적으로 html파일을 생성하지 못한다.
    -> 파일명을 webpack-manifest-plugin을 통해 json으로 받은 뒤, 해당하는 js파일명을 추출한뒤 컨트롤러에서 모델로 보내준다.

2. [문제] dynamic import시에 경로설정.
    -> 빌드된 js파일들을 서로 import할때 html페이지의 상위 path에서 파일을 불러온다.
        ex) "a/b/c" 페이지에서 0.js를 import 하면 "a/b/0.js" 를 호출한다.
    -> 해결방법 : WebMvcConfigurer에 path와 js위치를 매핑시켜준다.
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler(
			"/**",
			"/a/b1/**",
			"/a/b2/**",
			"/favicon.ico"
		).addResourceLocations(
			"classpath:/static/",
			"classpath:/static/",
			"classpath:/static/",
			"classpath:/static/favicon.ico"
		);
	}
}
```
    -> 페이지가 많아질경우 path마다 설정해줄수는 없다는 한계.


3. [문제] 일부 파일에서는 import().then()이 작동하지 않는다.
    -> *Uncaught (in promise) TypeError: n is not a constructor* 에러발생
    -> ES6 의 클래스 키워드가 원인? 
    -> UglifyJs 할때 ES5로 바꿔줘야하는데 이때 babelLoder가 작동하지않는다? 다른 파일에서는 (ES6)import가 적용되므로 문제가 아니다.
```javascript
 require.ensure([],function(require){
        const DomElement = require('../DomElement').default;
```
    -> 위와같이 수정하면 dynamic import 가 정상 동작한다.

4. [?] webpack 에서 gz파일로 압축하여 spring boot를 실행시키면 gz파일을 불러오지않고 js파일만 불러온다
    -> tomcat 에서 압축해서 보내는 방법 사용
    -> server.compression.enabled = true


