---
title: "[대규모 시스템 설계 기초] 14 유튜브 설계"
date: 2022-06-07 19:10:00 +0400
categories: design architecture study
tags: architecture
---
# 유튜브 설계
## 문제 이해 및 설계 범위 확정
### 2020 유튜브 통계
- 월간 능동 사용자수 20억
- 매일 재생되는 비디오수 60억
- 5천만명 창작자
- 80개언어 지원
- 모바일 인터넷 트래픽 37%를 유튜브가 점유
- 유튜브 광고수입은 2019년 150억달러
### 요구사항
- 비디오 업로드, 시청
- APP WEB SMART_TV
- DAU 500만
- 유저당 30분 평균 소비시간
- 다국어 지원
- 해상도 대부분 지원가능
- 암호화 필요
- 비디오 파일크기 제한 최대 1GB
- 클라우드 서비스 활용 가능. (낮은 인프라 비용)
### 개략적 규모 추정
- 한 사용자는 평균 5개의 비디오 시청
- 10% 사용자가 하루 비디오 1개 업로드
- 비디오 평균크기 300MB
- 매일 요구되는 새로운 저장 용량 500만 * 10% * 300MB = 150TB
- CDN 비용 : 500만 * 5비디오 * 0.3GB * 0.02$ (1GB당 CloudFront가격) = 150,000$
## 개략적 설계안 제시 및 동의 구하기
- **시스템 설계면접은 모든것을 밑바닥부터 만드는것과는 관계가 없다.**
- CDN과 Blob저장소는 기존 클라우드서비스를 활용한다.
- CDN : 비디오는 CDN에 저장한다. 재생버튼을 누르면 스트리밍 가능.
- API서버 : 스트리밍을 제외한 모든 요청
  - 피드 추천
  - 비디오 업로드 URL 생성
  - 메타데이터 DB와 캐시갱신
  - 사용자 가입 등
### 비디오 업로드 절차
- 원본저장소 blob : 이진 데이터를 하나의 개체로 보관하는 DB관리 시스템
> 이미지넣기 -  p.253
### 비디오 스트리밍 절차
- 스트리밍 프로토콜
  - MPEG-DASH (Moving Picture Experts Group-Dynamic Adaptive Streaming over HTTP)
  - Apple HLS (HTTP Live Streaming)
  - Microsort Smooth Streaming
  - Adobe HTTP Dynamic Streaming, HDS
- 프로토콜마다 지원하는 비디오 인코딩이 다르고 플레이어도 다르다.
## 상세 설계
### 비디오 transcoding
- bitrate : 비디오를 구성하는 비트가 얼마나 빨리 처리되어야하는지 나타내는 단위. bitrate가 높으면 고화질
- 상당수의 단말과 비디오가 특정 비디오포맷만 지원하므로, 호환성을 위해서라도 여러포맷으로 인코딩해야한다.
- 네트워크 대역폭에따라 화질을 조정하면 끊김없는 비디오재생이 지원된다.
- 원본비디오는 저장공간이 많이 필요하다.
- 컨테이너 : 비디오파일, 오디오 등 메타데이터를 담는 바구니같은것. (avi, mov, mp4 ..)
- 코덱 : 화질은 보존하면서 파일 크기를 줄일 목적으로 고안된 압축 및 압축 해제 알고리즘. (H.264, HEVC ..)
### 유향 비순환 그래프 모델 (DAG)
- 작업을 단계별로 배열할수있도록 하여 작업들이 순차적으로 또는 병렬적으로 실행될수있도록 하고있다. 
> kafka stream 같은건가??
> 그림 14-9
### 비디오 transcoding architecture
#### 전처리기
- 비디오 분할
  -  비디오 스트림을 GOP(Group of picture: 특정 순서로 배열된 프레임 그룹)로 쪼갠다. 
  - 하나의 GOP는 독립적으로 재생가능하며, 보통 몇초.
- DAG 생성
  - 클라이언트 프로그래머가 작성한 설정 파일에 따라 DAG를 만들어낸다.
- 데이터 캐시
  - 전처리기는 분할된 비디오 캐시이기도 하다. GOP와 메타데이터를 임시저장소에 보관한다.  인코딩이 실패하면 캐시를 활용해 인코딩을 재개한다.
#### DAG 스케줄러
- DAG 그래프를 단계별로 분할한다.
#### 자원 관리자
- 최적 작업(우선순위)/서버를 골라 실행지시.
#### 작업 서버
#### 임시저장소
- 비디오 프로세싱이 완료되면 삭제된다.
- 크기가 작으면  메모리에 캐시, 크면 Blob 저장소
> 그림 14-10
### 시스템 최적화
- 비디오 병렬 업로드 : GOP로 분할하여 업로드한다.
- 업로드 센터를 지역별 근거리에 지정.
- 모든 절차를 병렬화
  - 원본저장소 > 다운로드 모듈 > 인코딩 모듈 > 업로드 모듈 > CDN 모든 절차에 메시지큐로 도입하면 
- 안전성 최적화 : 미리 사인된 URL (?) = 접근 공유 시그니처
  - 허가받은 사용자만이 올바른 장소에 업로드하기위해, 이미 접근권한이 주어진 상태의 url을 반환한다
- 안전성 최적화 : 비디오 보호
  - DRM
  - AES : 암호화된 비디오는 재생시에만 복호화한다.
  - watermark : 비디오 위에 이미지 오버레이를 올리는것.
- 비용 최적화 : 
  - 인기 비디오만 CDN, 나머지는 비디오 서버로 재생
  - 인기 없거나 짧은 비디오라면 필요시 인코딩후 재생
  - 지역별 인기 비디오는 CDN 복제가 필요없다.
  - CDN 직접 구축
### 오류처리
- 회복 가능 오류 : retry 해보고, 안되면 오류코드 반환
  - 업로드 오류
  - 비디오 분할 오류 : GOP분할이 안되면 분할하지않고 전송.
  - transcoding 오류
  - 전처리기 오류 : DAG 재생성
  - ..
- 회복 불가능 오류 : 비디오 포맷이 잘못되었거나, 회복불가능하다면 비디오 작업 중단후 오류코드 반환
## 마무리
더 논의할 내용
- API 규모 확장성 확보 방향 : 무상태이고, 확장가능 어필.
- DB계층 규모 확장성 확보 방안 : 샤딩 방안
- live streaming 
  - 라이브는 응답 지연이 낮아야하므로, 스트리밍 프로토콜 선정에 유의해야한다.
  - 작은 단위의 데이터를 실시간으로 빨리 처리해야하기때분에 병렬성은 필요없을지도.
  - 오류처리방식이 달라야한다. retry하면 오래걸릴것.
- 비디오 삭제 : 부적절 컨텐츠 필터링, 신고 등. 
----

