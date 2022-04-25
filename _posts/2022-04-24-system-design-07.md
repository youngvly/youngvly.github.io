---
title: "[대규모 시스템 설계 기초] 07 분산 시스템을 위한 유일 ID 생성기 설계"
date: 2022-04-15 19:10:00 +0400
categories: 대규모 시스템 설계 기초
tags: architecture
---

# 분산 시스템을 위한 유일 ID 생성기 설계
- auto_increment를 쓰면, db가 여러대면 delay를 낮추기 힘들것.
> 왜 안좋은지 더 알아보자 
1. 문제 이해 및 설계 범위 확정.
  : 시스템 설계 면접 문제를 푸는 첫단계는, 적절한 질문을 통해 모호함을 없애고 설계방향을 정하는 것   
  - 요구사항
   - 유일id
   - 숫자 id
   - 64bit
   - 시간순 정렬 지원.
   - 초당 1만개 생성가능
2. 개략적 설계안 제시 및 동의 구하기.
 1) 다중 마스터 복제 (multi-master-replication)
   - db의 auto_increment를 쓰는데, 1이 아니라 현재 사용중인 db서버 수만큼 증가시킨다.
   - 규모 확장성 문제 해결가능, 초당 생산 가능 ID수도 늘릴 수 있다.
   - 여러 데이터 센터에걸친 확장은 어렵다.
   - ID 유일성은 보장되지만, 시간순 정렬을 보장할 수 없다. (서버마다 다름)
   - 서버 추가/삭제시 잘 동작하도록 만들기 어렵다.
 2) UUID (universally unique identifier)
 - 단순하고, 동기화가 필요없고, 규모확장이 쉽다.
 - ID가 128bit로 크고, 시간순 정렬이 불가, string이다.
 3) 티켓 서버
 - auto_increment기능을 갖춘 db를 한대만 사용.
 - 구현이 쉽고, 중소규모 에 적합.
 - 티켓서버가 SPOF가 된다. 티켓서버를 여러대 준비해야하고, 그럼 동기화문제가 다시 발생.
 - [Flickr](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)
 4)  트위터 스노우플레이크 접근법 (snow flake)
 - id의 구조를 여러 section으로 분할한다.
   -  sign bit : 음-양수 구별 등 언젠간쓸 값 (?)
   - timestamp : epoch millisecond 
   - datacenter ID : n bit =  $2^n$ 개의 데이터 센터 지원 가능.
   - 서버 ID : $2^n$ 개의 서버 사용 가능.
   - 일련번호 : ID 생성시마다 +1 / 1ms 시마다 0으로 초기화된다.
 > 이건 DB서버에 적용되는 알고리즘인가? 
3. 상세 설계
 - 타임스탬프는 41bit인데, 약 69년이다. overflow 시점을 기억하고있어야함.
 - 일련번호는 어떤 서버가 같은 ms동안 하나이상의 id를 만드는 경우에만 0보다 크다 > 예상 사용량에따라 bit수 조절하면 될듯
4. 마무리 / 추가 논의사항?
  - 시계 동기화 : 서버들이 전부 같은 시계를 사용해야하는데, 서버가 멀티코어라면 유효하지않을 수 있다.
  - 각 section의 길이 최적화 : 동시성이 낮고 수명이 길다면 일련번호 절을 줄이고, 타임스탬프를 길게 갈수도.
---
- [snowflake](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake)

> 애플리케이션 서버에서 유일 id 생성하는 방법은?
- [MongoDB](https://www.mongodb.com/docs/manual/reference/method/ObjectId/) timeStamp-processRandomValue-incrementingCounter
- 

