---
title: "대규모 시스템 설계 기초 2 : 7 호텔 예약 시스템"
date: 2024-10-29 17:00:00 +0900
categories: study, architecture
tags: study, architecture
---


# 1. 문제 이해와 설계범위 확정
## 질문
- 시스템 규모
- 대금 지불 시점 : 예약시
- 예약 인입 : 웹,앱으로만
- 다른 고려사항 : 10% 초과예약이 가능해야한다.
- 시간이 제한되어있으므로 검색은 제외합니다. (면접시 범위를 스스로 줄일수도있다!)
- 객실 가격은 유동적.

## 비기능 요구사항
- 높은수준의 동시성 
- 적절한 지연시간 : 요청 처리에 응답시간이 적당히 늦어도 허용됨.

## 개략적 규모 추정
- 5,000개의 호텔, 100만개의 객실
- 평균 객실 70%가 사용중이고, 평균 투숙기간 3일
- 일 예약건수 $100만 * 0.7 / 3 = 약 24만$
- 초당 예약건수 = $24/ 10^5 = 3$ TPS

### QPS
약 10% 사용자가 다음 단계로 진행한다고 가정.
1. 객실 예약 페이지 3 QPS(TPS)
2. 예약 상세 페이지 30 QPS
3. 객실 상세페이지 300 QPS

# 2. 개략적 설계안 제시 및 동의 구하기
## API 설계
- 신규 예약접수 API : POST `/v1/reservations`
	```json
	{
	startDate : 'yyyy-mm-dd',
	endDate : 'yyyy-mm-dd',
	hotelId : 123,
	roomId : 123A,
	reservationId : 123   // 이중예약 방지, 멱등성이 보장되는 키
	}
	```

## 데이터 모델
#### 서비스 특성
- 이벤트가 있는경우 트래픽 급증가능
- 평시 시스템 규모가 크지않음.
- R > CUD 10배? 크게 차이가 있는편이 아님.

RDB를 선택.
- 읽기빈도가 쓰기 연산에 비해 높은 작업 흐름을 잘 지원한다.
	- NoSQL은 대체로 쓰기연산에 최적화되어있다.
- ACID(원자성/일관성/격리성/영속성)을 보장한다.
	- 중복처리방지에 꼭 필요한 속성.
- 데이터 모델링과 호텔/객실 등 사이 관계를 나타내기 쉽다.
- 일반적인 예약에서 객실의 특정 호수를 예약하는것이 아닌, 객실 유형을 선택하므로 데이터 모델링시 고려되어야한다.

## 개략적 설계안
// 이미지 참고 p.238

# 3. 상세 설계
## 개선된 데이터 모델
- 호텔서비스
	- hotel
	- roomType
	- room (roomTypeId, hotelId)
- 요금 서비스
	- roomTypeRate : (hotelId, roomTypeId, date, rate) 날짜별 요금상태
- 투숙객 서비스 (회원)
	- guest
- 예약 서비스
	- roomTypeInventory :(hotelId, roomTypeId, date), total_inventory, total_reserved : 날짜별 수용가능한 객실수, 예약된 객실수
		- 데이터가 너무 많아지면 cold, hot으로 분리 혹은 샤딩으로 해결가능.
	- reservation 

### 용량 추정
$5000개의 호텔 * 20개의 객실유형(가정) * inventory는 2년간 저장 * 365d = 7,300만개$ * DB 복제 서버

## 동시성 문제
- 예약 API 요청파라미터에 미리 발급받은 멱등key를 추가하여 중복 처리를 방지한다.

### lock
- ACID의 Isolation 에 따라 각 트랜잭션은 상대 트랜잭션의 변경사항을 모른다. 동시 예약이 가능해질 수 있다. -> lock으로 해결
1. 비관적 lock
	- SELECT ... FOR UPDATE -> mysql 에서 레코드 락 걸림
	- 장점 :
		- 구현이 쉽고, 모든 갱신 연산을 직렬화하여 충돌을 막는다.
		- 데이터 경합이 심할때 유용
	- 단점 :
		- 여러 레코드에 락을 걸면 데드락이 발생가능.
		- 트랜잭션의 수명이 길거나 많은 엔티티에 관련된 경우 db성능문제.
1. 낙관적 lock
	- 버전번호, 타임스탬프로 현재 번호보다 큰 버전이 오지않으면 트랜잭션 중단.
	- 장점 :
		- db자원에 락을 걸지 않으므로, db성능 문제 없음.
		- 데이터에 대한 경쟁이 치열하지 않을때 적합. 
	- 단점
		- 경쟁이 치열할때는 재시도가 여러번 걸리면서 응답시간이 느려질 수 있음.
3. 데이터베이스 제약 조건
```sql
CONISTRAINT `check_room_count` CHECK((`total_inventory - total_reserved` >= 0))
```
	-  장점
		- 구현이 쉽다
	- 단점
		- 경쟁이 심하면 실패하는 요청 수가 많이 늘어나 사용자경험이 좋지않을 수 있다.
		- 지원되지않는 db로 교체시 이슈.
	
## 시스템 규모 확장
- DB 샤딩
- 캐시
	- 현재, 미래의 데이터만 중요하고 과거데이터는 중요하지않은 특성이 있다.
	- TTL, LRU로 캐시최적화 가능.

#### 캐시와 DB의 데이터 일관성
- DB CUD 시점에 캐시를 비동기로 업데이트.
	- CDC
	- 비동기 업데이트 사이의 데이터 불일치는 사용자 경험 영향을 고려해야함.

## 마이크로서비스 아키텍처에서의 데이터 일관성 문제에 대한 해결방안
- 마이크로아키텍처이므로, 각 서비스는 독립된 DB를 가져야한다.
- 2단계 커밋 (2PC)
	- 모든 노드(서비스)의 원자적 트랜잭션 실행을 보증.
- Saga
	- 각 노드의 트랜잭션을 하나로 묶어 실패하면 롤백 트랜잭션을 순차실행.


---- 
책에서 나오는것처럼 동시성 문제와, 마이크로 서비스라면 일관성문제가 가장 큰 이슈일듯.
- 이벤트성 트래픽
    - 선착순 이벤트가 있는경우, 순서가 중요하므로 예약 앞에 큐를 도입?
    - 아고다 같은곳은 예약 확정이 메일로 오는편. 호스트의 수동 확인도 있지만 비동기로 예약되어도 될듯.
- 동시성 문제
    - 예약 선점하는 시스템이 하나 더 있어야할듯.
        - TTL을 활용한 redis를 이용하여 멱등적 id 발급, 선점?
        - 사례를 찾아보자
