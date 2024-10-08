---
title: "대규모 시스템 설계 기초 2 : 6.광고 클릭 이벤트 집계"
date: 2024-08-02 19:00:00 +0800
categories: study, architecture
tags: study, architecture
---

# 6. 광고 클릭 이벤트 집계
- 디지털 광고 핵심 프로세스는 RTB(Real-Time Bidding), 실시간 경매로 광고가 나갈 지면을 거래한다.
## 1. 문제 이해 및 설계 범위 확정
### 질문
1. 입력 데이터의 형태
2. 데이터의 양
3. 조회 되어야할 쿼리
4. 고려해볼만한 엣지케이스
    - 이벤트 중복
    - 이벤트 처리 지연
    - 시스템 장애 및 복구
5. 지연 시간 조건
### 요구사항
#### 기능 요구사항
- 지난 M분간의 클릭 수 집계
- 매 분 가장 많이 클릭된 상위 100개 
- 다양한 속성에 따른 집계 필터링
- 데이터의 양은 페이스북이나 구글 규모
#### 비기능 요구사항
- 집계결과의 정확성이 매출에 직결되므로 중요
- 지연되거나 중복이벤트를 처리해야함
- 부분 장애는 감내할 수 있어야함.
- 전체 처리 시간은 최대 수분을 넘지 않아야함.
#### 개략적 추정
- DAU 10억명
- 하루에 1개의 광고를 클릭한다고 가정, 클릭 QPS = 10,000 ~ 최대 5배
- 클릭 이벤트 하나당 0.1KB의 저장 용량이 필요하다면, 일일 저장소 요구량은 100GB, 월간 3TB

## 2. 개략적 설계안 제시 및 동의 구하기




