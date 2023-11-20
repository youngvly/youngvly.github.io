---
title: "[대규모 시스템 설계 기초] 11 뉴스 피드 시스템 설계"
date: 2022-05-08 19:10:00 +0400
categories: design architecture study
tags: architecture
---
# 뉴스 피드 시스템 설계
## 문제 이해 및 설계 범위 확정
요구사항
- APP, WEB 모두 지원
- 유저가 업로드하고, 친구의 글을 볼수있어야한다.
- 시간 흐름 역순 노출
- 친구 최대 5천명
- 트래픽 하루 천만명
- 게시글에는 미디어 파일 포함 가능.
## 개략적 설계안 제시 및 동의 구하기
- 피드 발행 : 사용자가 포스팅하면 캐시와 DB에 기록한다. 친구의 뉴스피드에도 전송된다.
- 뉴스피드 생성 : 모든 친구의 포스팅을 시간 흐름역순으로 모아서 만든다.
### 뉴스피드 API
- 피드 발행 API (새글 쓰기)
  - 포스팅 저장 서비스(post service) : 새글을 DB, cache에 저장한다. 
  - 포스팅 전송 서비스(fanout service) : 새 포스팅을 친구의 뉴스피드에 push한다. (chache)
  - 알림 서비스
- 피드 읽기 API 
## 상세 설계
### 피드 발행 흐름 상세 설계
1. 웹서버
  - 인증
  - 특정 기간동안 한 사용자가 올릴 수 있는 포스팅수 제한 (rate limmit)
2. 포스팅 전송(fanout) 서비스
 - 쓰는 시점에 fanout-on-write = push
   - 뉴스피드를 읽는데 드는 시간이 짧다.
   - 친구가 많은 유저가 push할 경우, 시간이 많이 소요된다. (hotkey)
   - 서비스를 자주 사용하지 않는 유저의 피드도 갱신해야 하므로 자원 낭비.
   - twitter, follower가 많고, 트위터는 애초에 상대에게 메세지 보내는게 목적
 - 읽는 시점에 fanout-on-read = pull
   - 비활성화된 사용자, 서비스에 거의 로그인하지 않는 사용자의 경우 유리.
   - 뉴스피드를 읽는데 많은 시간 소요. 
   - facebook, TAO(The associated and object)
 - push and pull
   - 대부분의 사용자는 push, hotkey의 경우 pull 
 - 뉴스피드 cache
   - post_id : user_id 순서쌍을 보관하는 매핑테이블,
   - 메모리 크기를 제한하여 오래된 스토리는 삭제. (최신성이 요구되기때문에, 오래된글까지 들고있을 필요는 없음)
3. 친구 DB
 - graph db는 친구관계, 친구 추천을 관리하기 적합하다. 
### 피드 읽기 흐름 상세 설계 
1.  뉴스피드 서비스는 포스팅 ID 목록을 가져온다.
2. 사용자 이름,포스팅 컨텐츠 등 각각의 캐시에서 가져와 완전한 뉴스피드를 만든다.
3. 캐시구조
![429AEBDC-1EC1-4511-B934-DF8F48E33E9F](https://user-images.githubusercontent.com/19251378/168613073-7bf8bc26-e6ec-4ee2-b13f-9ff4c3895522.jpeg)
  - 뉴스 피드 : 뉴스 피드의 id 보관
  - 콘텐츠 : 포스팅 데이터 보관, 인기 컨텐츠는 따로
  - 소셜 그래프 : 사용자간의 관계 정보
  - 행동 : 좋아요, 답글 등 보관
  - 횟수 : 좋아요,응답, 팔로워수, 등 정보 보관

## 마무리
- 더 다룰만한 주제
  - DB 규모확장
  - 수직적 vs 수평적
  - SQL vs NOSQL
  - master-slave 다중화
  - replica 에 대한 읽기 연산
  - consistency model
  - sharding
- stateless web tier
- 가능한 많은 데이터 캐싱할 방법
- 메세지 큐

> 규모산정이나 많이 생략된듯? 피드 읽기시에 hotkey인 유저는 어떻게 따로 가져올 것인가?
#### Cassandra
- 쓰기속도가 빠름
- 클러스터 내의 노드가 죽어도, 서비스에는 영향을 주지 않고 노드 추가와 같은 운영도 쉽다.
 
----
- [설계 상세](https://github.com/donnemartin/system-design-primer/tree/master/solutions/system_design/web_crawler)
- [social network service architecture d2](https://d2.naver.com/helloworld/551588)
- [line d2](https://d2.naver.com/helloworld/809802)


