---
title: "[대규모 시스템 설계 기초] 10 알림 시스템 설계"
date: 2022-05-08 19:10:00 +0400
categories: 대규모 시스템 설계 기초
tags: architecture
---
# 알림 시스템 설계
## 문제 이해 및 설계 범위 확정
요구사항
- 알림의 종류 : SMS, push, email
- 실시간 시스템, 약간의 지연은 허용
- 단말 : IOS, android, pc
- 알림 생성 : 클라이언트 애플리케이션, 서버측 스케줄링
- 유저의 알림 설정 가능.
- 하루에 1000만건 push / 100만건 SMS / 500만건 
## 개략적 설계안 제시 및 동의 구하기
### 알림 유형별 지원 방안
1. PUSH
 - 알림 제공자(provider), 알림 요청을 만들어 apns로 보내는 주체
   - 단말 토큰, 알림내용 payload
 - (IOS) APNS : 애플이 제공하는 푸시알림 원격 서비스   
 - (Android) FCM 
 - 단말
2. SMS 메세지
 - 보통 제3서비스를 사용한다 > 유료 : twilio, nexmo 
3. 이메일
 - 고유 이메일 서버 혹은 상용 서비스 : Sengrid, Mailchimp
### 연락처 정보 수집 절차
- 앱 설치 혹은 계정 등록시 정보 수집. (단말 토큰, 전번, 이메일 주소 등)
- DB 설계, user:device = 1:n
### 알림 전송 및 수신 절차 
- 개략적 설계안 1
 - 여러 서비스에 지원가능한 알림 전송/수신처리 API 서버
 - 제3자 알림제공 서비스의 규모 확장성 (apns, fcm, jpush..)
- 문제
 - SPOF : 서버가 1개면 장애에 위험하다
 - 규모 확장성 : db, cache등 컴포넌트 규모를 개별적으로 늘릴 방법 필요. 
 - 성능 병목 : 모든걸 한시에 보내려하면, 시스템 과부하 가능. 
- 개략적 설계안 2
 - db, cache를 주서버에서 분리한다.
 - 자동으로 수평적 규모확장이 가능하도록 한다.
 - 메시지 큐를 두어 시스템 컴포넌트 사이의 강결합을 끊는다. (각 알림 유형별로 큐를 두어 병렬 분리)
  -  다량의 알림전송에 대비하여 버퍼역할도 된다.
## 상세 설계
### 안정성
1. 데이터 손실 방지
 - 지연 및 순서는 틀려도 괜찮지만, 소실은 안된다. 
 - 알림 데이터를 db에 보관하고, 재시도 메커니즘을 구현해야한다. (알림 로그 db)
2. 알림 중복 전송 방지
 - 완전 방지는 불가능하겠지만(참고문헌), 이벤트id를 중복검사하여 방지한다. 
### 추가로 필요한 컴포넌트 및 고려사항
1. 알림 템플릿
  - parameter, tracking link등을 조정가능.
2. 알림 설정
  - 유저별 알림 설정기능, 특정 종류의 알림을 보내기 전에 알림 켜두었는지 확인.
3. 전송률 제한
  - 한 유저당 알림 빈도제한, 
4. 재시도 방법
 - 재시도 전용 큐에 넣고, 개발자애게 알림 등.
5. 푸시 알림과 보안
  - 3자 알림서비스별 보안키 관리
6. 큐 모니터링
  - 큐에 쌓인 알림 갯수가 많으면 서버 증설을 고려
7. 이벤트 추적
  - 알림 확인율, 클릭율, 실제 앱 사용으로 이어지는 비율 등
### 개선된 설계안
## 마무리
더 집중한 주제
- 안정성 reliability
- 보안 security
- 이벤트 추적 및 모니터링
- 사용자 설정
- 전송률 제한
- 
----
- [You cannot have exactly once delivery](https://bravenewgeek.com/you-cannot-have-exactly-once-delivery/)
  - producer - broker 발송중에 broker가 죽는 등의 이유로 발송에 실패한다면? 메세지 증발, at-least-once 불만족
  - consumer 처리중에 ack만 못보내고 서버가 죽는다면? consumer가 동일 메세지를 다시 소비할것, at-most-once 불만족 (중복)
- [FLP](https://levelup.gitconnected.com/practical-understanding-of-flp-impossibility-for-distributed-consensus-8886e73cdfe5) 분산 환경에서 단일 노드의 장애를 감내할수있는 합의알고리즘(zookeeper,,) 은 없다???
- [유사 Notification 설계](https://medium.com/interviewnoodle/notification-system-architecture-e0d98ab3d18)
![image](https://rajivcloudzonecom.files.wordpress.com/2021/08/blog-notification-systemdesign-1.jpeg)


