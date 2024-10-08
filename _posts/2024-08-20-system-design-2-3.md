---
title: "대규모 시스템 설계 기초 2 : 3 구글 맵"
date: 2024-08-29 17:00:00 +0900
categories: study, architecture
tags: study, architecture
---

# 3. 구글 맵

## 1. 문제 이해 및 설계 범위 확정
### 기능 요구사항
- 사용자 위치 갱신
- 경로안내 서비스 (ETA 포함)
- 지도표시
### 비기능 요구사항
- 경로 안내 정확도
- 화면에 부드러운 경로 표시
### 지도 101
- 지오코딩 : 주소지를 위도 경도로 변환하는것.
- 지오해싱 : 분할된 사분면을 해시값에 매핑시키는것.
- 지도표시 : 지도 확대수준에 근거하여 적합한 지도타일 사이즈를 랜더링.
- 경로 안내 알고리즘을 위한 도로 데이터 처리 : 대부분 Djikstra 혹은 A* 알고리즘. 교차로 = node, 도로 = edge 로 표현한 그래프 자료구조를 가정한다.
- 계층적 경로 안내 타일 : 넓은 지역 범위를 높은 정밀도의 타일로 경로탐색하면 비효율적이다. 
    - 보통 지방도와 고속도로 처럼 구분되는 세가지의 구체성 정도를 구분하여 세 종류의 경로안내 타일을 준비한다.
### 개략적 규모 측정
- DAU 10억
- 유저당 평균 사용시간 35분/week
- 평균 하루 50억 min 사용된다. = 300만 QPS
- 저장공간
    - 세계지도를 21번 확대하려면 4^21 = 4.4조개의 타일이 필요하다.
    - 타일 한장이 100KB => 확대수준 21에서는 440PB => 비 거주지역이 90%이고, 높은 비율로 압축할 수 있으므로 440 * 0.9 =~ 50PB 라고 가정한다.
    - 21번 확대 + 20번 확대 + ... 1번 확대 = 50 + 50/4 + 50/4^2 =~ 67PB
- 서버 대역폭
    - 클라이언트는 위치 변경내역을 배치로 15s마다 서버로 보낸다. = 300만 QPS/15 = 20만
    - 최대 QPS는 평균치의 5배 = 1백만


## 2. 개략적 설계안 제시 및 동의 구하기
### 위치서비스
- 사용자는 15s마다 변경된 위치정보를 서버에 보낸다.
- 서버는 많은 쓰기 요청을 해야하므로, 확장성있는 DB에 데이터를 저장한다. (Casandra)
- 서버와 클라 통신 프로토콜은 HTTP keep-alive 를 활용할 수 있을것.

### 경로 안내 서비스
- input : 출발지, 목적지
- output : 거리, 소요시간, 중요한 경로는??책이라 생략된건지 p.88

### 지도 표시
1. 클라이언트가 보는 지도의 확대 수준에 근거하여 지도 타일을 서버가 실시간으로 만드는 안.
    - 서버에 심각한 부하, 캐싱불가
    > 이게 이미지인데 즉석에서 만드는게 가능한건지?
2. 지오해시별 URL을 CDN에 저장하여 캐싱.
    - 필요한 지오해시는 서버에서 연산. 클라이언트단에서 연산하게 되면 앱이 무거워지고, 업데이트 어려워지고 등등 
    > p.91 지도 하나를 랜더링하는데 여러 지오해시가 필요하다면, 네트워크 트래픽이 비효율적 이어 보이는데, 그럼에도 지오해시별 CDN을 타는게 득인건가? 비용문제?

## 3. 상세 설계
### 데이터 모델
- 경로 안내 타일
    - 교차로(node), 도로(edge) 인 그래프 데이터는 메모리에 인접 리스트 형태로 보관하는것이 일반적. 메모리에 다 올릴 수없으므로, S3같은 객체저장소에 인접리스트->bynary file 보관을 병행.
- 사용자 위치 데이터
    - 유저의 타임스탬프별 위치 데이터는 실시간 교통상황, 경로 갱신 등 유용하게 사용될 수 있다.
    - 많은 쓰기연산엔 카산드라 , NoSQL
    - p.97 CAP 정리 : 일관성 / 가용성 / 분할내성 모두 만족시킬순 없다. 
        - 여기서는 일관성의 비중이 작다 : 일부가 누락되어도 된다?
    - PK = user_id, timestamp
        - partitioning key : user_id
        - clustering key : timestmap
- 지오코딩 DB
    - 읽기연산이 쓰기연산보다 훨씬 많으므로 레디스같은 Key-value 저장소
- 미리 만들어둔 지도 이미지
    - 클라이언트에서 cdn을 통해 받아 랜더링한 지도이미지는 캐싱하는것이 효율적.

### 서비스
- 사용자 위치 데이터는 어떻게 이용되는가
    - 실시간 교통상황
    - 지도 데이터 정확성 개선 소스
    - 신규/폐쇄된 도로 감지
    - 여러 소스로 사용하기위해 kafka 메세지로 발행하여 각 용도에 맞는 컨슈머가 처리하도록 한다.
- 지도 표시
    - 최적화 : 벡터 사용 + WebGL
        - 이미지 전송 대신 경로, 다각형 등의 벡터 정보를 보내고 클라이언트가 지도를 그려내는 방식
        - 월등한 압축률!
        - 이미지보다 매끄러운 지도 확대 효과
- 경로 안내 서비스
    1. 지오코딩 서비스
        - 출발지, 목적지의 위도+경도 정보를 조회한다.
    2. 최단 경로 서비스
        - 도로 상황은 고려하지않고, 도로 구조만 의존하여 그래프 탐색.
        - 정적 데이터이므로 경로를 캐싱할 수 있다.
        > 그래프 DB정도로 저장된거라면, 타일이 필요한가? 노드정보만 내려줘도 되지 않남 경로안내타일이 어떻게 생긴건지 아직 이해가 안됨.
    3. 예상 도착시간 서비스
        - 현재 교통 상황, 과거 이력에 근거하여 계산.
    4. 순위 결정 서비스
        - 유료도로 제외, 고속도로 제외 등 필터링.
    - 중요정보 갱신 서비스
        - 그림 3.15의 실시간 교통정보 서비스, 경로안내 타일 처리 서비스 (신규/폐쇄 도로 싱크)
    - 적응형 ETA, 경로변경
        - 특정 타일에서 사고가 났다고 가정하면, 현재 경로안내를 받은 유저들에게 어떻게 전파할것인가
            1. 경로안내를 받는 유저들과 경로 안내 타일 정보를 서버에서 들고있다고 가정.
                - user_n : r1, r2, r3, rm
                    - 영향받는 유저 탐색 시간 = 유저 n명 * 유저당 평균 타일수 m = O(n*m)
                - user_n : r1, super(r1) , super(super(r1)), r2, super(r2), super(super(r2)) : 재귀적으로 더 큰 타일을 같이 저장한다.
                    - 사용자의 레코드 마지막 타일에 이슈타일이 있으면 영향받는 유저. = O(n)
                    > 왜 마지막 타일이 이슈타일??? 
                - 이 방법은 도로상황이 다시 개선되었을때 재계산할 수 없음.
            2. 클라이언트단에서 ETA 재계산을 주기적으로 요청.
                - 전송 프로토콜
                    - 푸시알림, long polling, webSocket, SSE