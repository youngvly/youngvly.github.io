---
title: "[대규모 시스템 설계 기초] 04 처리율 제한 장치의 설계"
date: 2022-04-11 19:10:00 +0400
categories: design architecture study
tags: architecture
---
# 04 처리율 제한 장치의 설계
1. 어디에  둘 것인가
  - 클라이언트측 : 위변조가 가능해서 통제가 어려울수 있다.
  - 서버측 : 서버별로 설계가 필요하고, 다중서버라면 어려울 수 있다.
  - 미들웨어 :  api gateway에 포함될 수 있다.
2. 알고리즘
  1) 토큰 버킷 (token bucket)
    : 버킷에 토큰을 주기적으로 발급하고, 요청별로 토큰을 소모하며, 없으면 요청이 무시된다.
    - 구현이 쉽다.
    - 메모리에 효율적이다.
    - 짧은시간 집중되는 트래픽처리가 가능하다.
    - 버킷 크기와 토큰 공급률을 튜닝하기 어렵다.
  2) 누출 버킷 (leaky bucket)
    : 토큰 버킷 알고리즘과 비슷하지만, 요청 처리율이 고정되어있다. 큐에 요청을 넣고(FIFO) 고정속도로 처리.
  3) 고정 윈도 카운터 (fixed window counter)
    : 타임라인을 고정된 간격의 window로 나누고 카운터를 붙여 제한한다.
    - 윈도 경계부근에 순간적으로 많은 트래픽이 몰리면 더 많은 요청이 처리될 수 있다. (윈도우를 어떻게 분리하느냐에 따라 기대치보다 많은양을 처리한 결과가 나올 수 있다.)
  4) 이동 윈도 로그 (sliding window log)
    : 들어오는 요청의 timestamp를 기준으로 윈도를 계산하며, 거부된 요청도 타임스탬프를 기록하고,  만료된경우 삭제한다.
    1분 윈도우 -> 1:25:00 요청도착 = (0:25:00~1:25:00 window)
     - redis sorted set같은 캐시에 보관하고, 메모리를 많이 소요한다.
     -  고정윈도 카운터의 요청이 몰리는 단점을 해결할 수 있다.
  5) 이동 윈도 카운터 (sliding window counter)
    : 현재 1분간의 요청수 + 직전 1분간의 요청수 x 이동 윈도와 직전 1분이 겹치는 비율
    - 메모리 효율이 좋다. 요청이 균등하다는 가정으로 계산되는 값이기때문에 다소 느슨하다.
    - 고정 윈도 카운터와 이동 윈도 로깅 알고리즘이 결합된 것.
3. 구현
  1) 경쟁조건 : 캐시에 카운팅시 동기화이슈로 락을 걸어야한다. 
     - 대안 lua script / sorted set **알아보기**
  2)  동기화 이슈 : 여러대의 미들웨어를 두면, 고정세션(비추) 혹은 중앙 집중형 데이터 저장소 사용으로 해결.
  
4. 추가
  1) 클라이언트 설계는 어떻게해야 효과적일까
    - 클라이언트측 캐시
    - 재시도 구현시 back-off 시간을 둔다.
  2) 성능 최적화
    - 데이터센터별로 트래픽전달.
    - 동기화시 최종 일관성 모델 사용 > **6장 데이터 일관성 항목 참고 **
    - 
---
### Redis sort sets
https://engineering.classdojo.com/blog/2015/02/06/rolling-rate-limiter/
https://codepool.github.io/articles/rate-limiting-using-redis-lists-and-sorted-sets
> Using Lua makes sure that no other redis client can execute the script parallelly.
### Lua script
redis에 script자체를 넘겨서 set/get등등 한개의 transaction안에서 되는것으로 보임.
### Rate limit Request with iptables (OSI 3th layer)  
https://blog.programster.org/rate-limit-requests-with-iptables
### 정리굿
https://www.mimul.com/blog/about-rate-limit-algorithm/

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
    
4. 