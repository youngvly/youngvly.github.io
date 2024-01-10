---
title: "[Http 완벽 가이드] 4 커넥션 관리"
date: 2023-11-22 19:10:00 +0400
categories: http, study
tags: network, http
---
# 1. TCP 커넥션
- 각 TCP 세그먼트는 하나의 IP주소에서 다른 IP주소로 IP 패킷에 담겨 전달된다. 
- IP 패킷 구성요소
  - IP 패킷 헤더 (20byte)
  - TCP 세그먼트 헤어 (20byte)
  - TCP 데이터 조각 (0~n byte)
- TCP 커넥션의 식별값 
  - 발신지 IP주소
  - 발신지 port
  - 수신지 IP주소
  - 수신지 port
- 소켓 API는 HTTP 프로그래머에게 TCP와 IP의 세부사항들을 숨기는 인터페이스이다.
  - bind : 소켓에 로컬 포트 번호와 인터페이스 할당
  - connect : 로컬의 소켓과 원격의 호스트 및 포트사이에 TCP 커넥션을 생성
  - ...
# 2. TCP 의 성능에 대한 고려

## HTTP 트랜잭션 지연
- DNS 지연
   - 요즘 대부분의 HTTP 클라이언트는 최근 접속한 사이트에 대한 IP주소의 DNS 캐시를 가지고있다.
- 대부분의 HTTP 지연은 TCP 네트워크 지연때문에 발생한다.
## TCP connection Handshake 지연
- SYN -> SYN + ACK 단계에서 발생하는 지연,
- HTTP 프로그래머들은 이 패킷들을 보지 못한다. 패킷들은 보이지 않게 TCP 소프트웨어가 관리하기때문.
- 크기가 작은 HTTP트랜잭션은 50%이상의 시간을 TCP를 구성하는데 쓰일 수 있다.
- 

## 확인응답 지연 Piggybacking

- 참고 : https://www.geeksforgeeks.org/piggybacking-in-computer-networks/
## TIME_WAIT 의 누적과 포트고갈

- 소켓 옵션에서, TW 커넥션 재사용 옵션이 꺼져있으면 BindException이 발생할 수 있다
  - 참고 : https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=sdug12051205&logNo=221055024751