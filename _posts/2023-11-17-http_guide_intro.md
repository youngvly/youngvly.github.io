---
title: "Http 완벽 가이드"
date: 2023-11-17 19:10:00 +0400
categories: http, study
tags: network, http
---


 # 1. 웹의 기초
 - MIME : Multipurpose Internet Mail Extensions : 멀티미디어 타입
   - 주타입/부타입 포맷  ex) text/html , image/jpeg
 - URI : Uniform resource identifier 
   - URL + URN 통합 개념
 - URL : Uniform resource locator (URI와 URL은 보통 같은의미로 사용된다)
   - 리소스가 어디있는지를 설명해서 리소스를 식별한다. (urn은 이름으로 식별)
 - URN : Uniform resource name
   - 리소스의 위치에 영향받지않는 유일무이한이름. 어떤 네트워크 프로토콜을 사용해도 동일한 리소스가 반환되어야한다.

- 텔넷 유틸리티는 키보드를 목적지로 TCP 포트로 연결해주고, 출력 TCP 포트를 원격화면을 출력해준다.
```sh
$ telneet www....com 80
~
$ GET /abc.html HTTP/1.1
$ HOST: www....com
~

# 응답
<HTML> ...<HTML>
```
- nc (netcat)은 HTTP를 포함한 UDP 혹은 TCP기반의 트래픽을 조작하고 스크립트할 수 있게 해준다.
  
## 웹의 구성요소
- 프락시 : 클라~서버 사이 위치한 HTTP 중재자, 주로 보안 용도
- 캐시 : 자주 조회되는 리소스의 사본을 저장하는 특별한 종류의 HTTP 프락시 서버
- 게이트웨이 : 다른 서버들의 중재자 서버, 주로 HTTP 트래픽을 다른 프로토콜로 변환하기위해 사용.
- 터널 : 데이터를 열어보지않고 그대로 전달해주는 HTTP 애플리케이션
  - 암호화된 SSL 트래픽을 HTTP 커넥션으로 전달하여 웹트래픽만 허용하는 사내방화벽을 통과시키는 예
- 에이전트 : 웹 요청을 만드는 애플리케이션은 뭐든 ex) 브라우저, 봇


# 2. URL과 리소스

- URL 문법
  - `<schema>://<user-name>:<passwd>@<host>:<port>/<path>;<parameter>?<query>#<fragment>`
  - parameter : 특정 스킴에서 입력 파라미터를 기술하는 용도
  - query : 스킴에서 애플리케이션에 파라미터를 전달하는데 쓰인다. &로 구분
  - fragment : 서버로 전달되지는 않고, 브라우저에서 처리한다. ex) 스크롤이동
- 단축 URL (상대경로)
    1. 기저 url 스킴 상속받는경우
    2. html 에서는 기저 url을 가리키는 `<base>` 태그 사용가능
- 안전하지 않은 문자
  - 안전한 문자로 제공하고자 US-ASCII에서 지원하는 문자만 허용, 그외는 인코딩 필요
  - 인코딩은 %12 같은 ASCII 코드로 표현되는 두개의 16진수 숫자로 이루어진 문자
  - URL에서 예약어로 사용되는 문자의 경우 인코딩이 필수 -> java URL.encode()
  > spring에서 parameter는 디코딩해서 들어올까??
- URN은 어떻게?
  - 대략 프록시 방식으로 보임. resource resolver가 실제 리소스위치와 고정 Path를 매핑해주는 방식

# 3. HTTP 메서드
- 메세지 흐름
  - 인바운드 : 서버방향
  - 아웃바운드 : 사용자 에이전트 방향
  - HTTP 메서드는 다운스트림으로 흐른다. client -> proxy1 -> proxy2 -> [server] -> proxy2 -> proxy1 -> client
    - 요청에서 proxy1은 server의 업스트림, 
    - 응답에서 proxy1은 server의 다운스트림
- 메세지에서 버전번호는 어떤 애플리케이션이 지원하는 가장 높은 HTTP 버전을 가르킨다. 
  - 응답에서 HTTP/1.1 은 응답을 보낸 애플리케이션이 HTTP/1.1까지 이해할 수 있다는 의미
- 헤더의 컨텐츠가 길면 추가줄 앞에 하나의 스페이스 혹은 탭문자를 포함하여 줄내림을 할 수 있다.
## 메서드
- HEAD : HTTP/1.1 을 준수하기 위해서는 HEAD 메서드가 반드시 구현되어야한다.
- PUT : 새문서를 만들거나, 이미 존재한다면 교체하는 용도
- POST : 입력데이터를 서버에 전달하기 위한 용도 (?)
- TRACE : 주로 진단을 위해 사용되며, proxy를 거쳐 서버에 도달하는 과정을 추적, 본문없이 헤더에 포함되어야한다.
## 상태코드
- 100~199 : 정보성 상태코드
  - 100 Continue : 클라이언트는 나머지 요청을 이어서 보내야함을 의미, 
    - client : header Expect 100-continue 를 포함해서 요청해야한다.
    - server : 100 으로 응답. 
    - 의미가 애매해서 잘 사용되지 않음.
- 200~299 : 성공 상태코드
  - 200 : 상태
  - 201 created : post,put의 성공응답, 생성된 리소스에 대한 구체적 참조가 담긴 location 헤더를 포함해야한다.
  - 202 accepted : 요청은 받아들여졌으나, 서버는 아직 어떤 동작도 수행하지 않은 상태
  - 206 partitial content : 클라이언트가 Range 헤더를 포함하여 요청한경우, 서버는 Content-Range, Date 등의 헤더를 반드시 포함하여 범위요청 성공응답을 보내야한다.
- 300~399 : 리다이렉션 상태 코드
  - 302 Found : HTTP/1.0 에서 클라이언트는 Location 헤더값으로 GET 요청을 보낼것.
  - 303 SeeOther : HTTP/1.1에서 클라이언트는 Location 헤더값으로 GET 요청을 보낼것. (POST 요청이라도)
  - 307 Temporary Redirected : HTTP/1.1 에서 302,307 혼란을 막기위해 장려하는 코드.
  - 서버는 적절한 리다이렉트 코드 사용을 위해 클라이언트 버전 검사가 필요하다.
- 400~499 : 클라이언트 에러상태 코드

