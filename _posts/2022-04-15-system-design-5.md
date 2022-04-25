---
title: "[대규모 시스템 설계 기초] 05 안정 해시 설계"
date: 2022-04-15 19:10:00 +0400
categories: 대규모 시스템 설계 기초
tags: architecture
---

# 05 안정 해시 설계
## 해시키 재배치 문제
- 일반적으로 서버에 분산저장할때, `index = hash % serverCount` 로 계산.
- 서버한대가 죽으면? 인덱스계산 오류로 조회조차 불가하게될 수 있음.
## 안정해시
- 해시 테이블 크기가 조정될 때 평균적으로 keyCount/slotCount 개의 키만 재배치하는 기술.
- 일반적으로는 전체키를 재배치해야함.
- ![image](https://miro.medium.com/max/1400/1*Q7bGMpsvDWcCaOPvH8vICQ.jpeg)
- 방법
   - 서버와 키를 균등분포 해시 함수를 사용해 해시 링에 배치한다.
   - 키의 위치에서 링을 시계방향으로 탐색하다 만나는 최초의 서버가 키가 저장될 서버이다.
- 단점
   - 서버가 추가/삭제되면 파티션크기가 균등하지 않을 수 있다.
   - 키가 파티션에 고르게 분포되지 않을 수 있다.
### 가상 노드
- 키가 파티션에 비교적 고르게 분포되도록, 가상노드를 두어 하나의 서버가 여러 노드/파티션을 담당해야한다.
- 가상노드의 갯수를 늘리면 점점 분포는 균등해진다.
- ![image](https://miro.medium.com/max/1276/0*mUO6SwgTRKKwpjUL)

### 사용처
 - [cassandra](https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf)
    - 그냥 대충 사용한다는 내용
 - memcached
--- 
참고
- [안정해시](https://medium.com/geekculture/consistent-hashing-99ea4caaa24d)
