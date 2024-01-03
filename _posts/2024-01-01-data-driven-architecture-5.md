---
title: "데이터 중심 아키텍처 : 5.복제"
date: 2024-01-01 16:00:00 +0400
categories: study, architecture, db
tags: study, architecture, db
---

# 05. 복제
1. single leader
2. multi leader
3. leaderless
## 리더와 팔로워
- leader와 다중 복제서버 (replica)구조
### leader-based replication = active(능동) / passive(수동) = master/slave 복제
    - leader 노드에서는 write작업을 전담하여 변경을 로컬에 저장하고, replicationlog, change stream을 팔로워에게 전송하여 복제한다.
#### 동기 vs 비동기
- 동기 : leader 변경작업시 follower의 변경을 보장
    -  동기 노드가 죽으면 write작업 불가.
    - 일관성 있는 데이터 복사본을 가질 수 있다.
- 비동기 : leader는 변경작업 메세지를 follower에게 전송하지만, 응답을 기다리지는 않는다.
    - 약간의 지연이 발생.
- 반동기식 : leader - sync replica (1) - async replica (n)
    - 한개의 동기노드가 리더가 죽었을때 대체가능하다
### 새로운 팔로워 설정
중단없이 추가하는방식
1. leader의 snapshot을 신규 팔로워 노드에 복제한다.  
2. snapshot 이후 (binlog coordinate) 발생한 모든 데이터 변경을 복제한다.
3. 이제 다른 팔로워와 마찬가지로 동작 가능.
### 노드 중단 처리
#### 팔로워 장애 : 따라잡기 복구
- follower 에서 서버가 다운되기 전 마지막 트랜잭션을 기준으로 변경사항을 leader에게 요구하여 sync.
#### 리더 장애 : 장애 복구 (failover)
1. leader가 단순 타임아웃으로 끊어진것인지, 실제 서버장애인지 판단.
2. 새로운 leader 선출
    -  새로운 리더로 가장 적합한 노드는 이전 리더의 최신 데이터 변경사항을 가진 복제서버, 모든 노드에게 election(선출) 과정을 거친다. 
> kafka leader electon 도 이런 과정을 거치는건가?
3. 새로운 리더 사용을 위해 시스템을 재설정 한다. 
- 비동기식 복제를 사용한다면, new leader와 old leader간의 불일치가 발생할 수 있다. (충돌하는 쓰기), 이때 사이의 간극을 폐기하는 방식이 간단한데, 다른 db와 sync되어야하는 경우 어려워질 수 있다.
    - ex) MySql Auto increment PK가 Redis의 key로 사용되고있는 경우, PK가 중복되어 데이터 정합성이 깨질 수 있다.
- split brain : 특정 결함 시나리오에서 두 노드가 모두 자신이 리더라고 믿는 상황. 이때 데이터가 유실되거나 오염될 수 있어 위험한데, 둘중 한노드를 종료하는 매커니즘도 있다.
- 
### 복제 로그 구현
#### stagement-based replication 구문 기반 복제
- 모든 statement를 기록하고, follower에게 전송한다. (insert/update,,)
    - now()나 rand()는 다른 값을 생성시킬 수 있다. > 고정값 반환으로 대체
    - 데이터에 의존하는 업데이트인경우, 순서가 보장되어야한다.
    - 부수효과 trigger등을 포함하는 구문은 각 복제서버에서 다른 부수효과가 생길 수 있따.
- Mysql 5.1 이전버전에서는 사용되었지만, 위 문제들로 row-based replication으로 변경되었다.
#### 쓰기 전 로그 배송
- 로그 기반 복제방식은 동일 한 구조의 데이터 복제본을 만들 수 있지만, 저장소 엔진과 밀접하게 엮이게 되어 리더와 팔로워가 다른 버전의 sw를 사용할 수 있다.
    - 다른 버전 사용이 허용된다면, 팔로워가 더 높은 버전을 사용하고, 리더가 나중에 업그레이드 하여 무중단 버전업이 가능하다. 
#### 논리적 (row 기반) 로그 복제
- 논리적 로그(logical log) = 저장소 엔진의 물리적 데이터 표현과 구별되는 로그
    - 여러 row를 수정하는 트랜잭션은 여러 로그로 레코드 생성후 트랜잭션 커밋됨을 레코드에 표시한다.
- follower와 leader가 다른 버전의 엔진을 사용할 수 있고, 외부 애플리케이션이 파싱하기 쉬워서 change data capture에서 사용된다.
#### 트리거 기반 복제
- 원하는 스크립트를 트리거 형식으로 실행가능, 유연성이 좋아 많이 사용됨.
### 복제지연 문제
- read-scaling 아키텍처 : 팔로워간의 읽기요청을 분산하는 옵션
-  
  
 


