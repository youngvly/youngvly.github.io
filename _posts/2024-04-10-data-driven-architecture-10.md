---
title: "데이터 중심 아키텍처 : 10.일괄처리"
date: 2024-04-10 16:00:00 +0400
categories: study, architecture, db
tags: study, architecture, db
---

# 10. 일괄처리
### 정렬대 인메모리 처리
- 데이터양이 많아지면 작업을 위해 할당받아야할 메모리가 커질거고, 부족할것, 이때는 정렬 접근법을 사용해야한다. 
- 메모리에서 일부 정렬 -> 디스크로 세그먼트 저장 -> 최종으로 전체 메모리에서 병합 후 정렬.
- unix의 sort 가 이 방식을 사용하며, 대용량 파일도 병목없이 처리가능하다.
- 
### 유닉스 철학
1. 한 명령어는 한가지 일만 한다.
2. 결과가 다른 명령어 입력값으로 사용될 수 있으므로, 입력,출력은 명료해야한다. (이진 X)
3. 빠른 프로토타이핑 : 소프트웨어를 빠르게 써볼 수 있게 설계, 구축하라.
4. 자동화 : 프로그래밍 작업을 줄이려면 도구를 써라.

### 로직과 연결의분리
- 느슨한결합 : stdin, stdout으로 연결되어있다. 중간 데이터는 인메모리로 처리됨

### 투명성
- 파이프라인중 필요한 부분에서 결과를 찍어볼 수 있으므로, 투명성이 있고 실험용으로 사용하기 좋다.

## 맵 리듀스와 분산 파일 시스템
- 유닉스와 다른점은, 인풋 아웃풋을 분산파일시스템에 저장한다는것.
- 하둡의 HDFS는 비공유가 원칙으로, 중앙 디스크 저장을 사용하지않는다. NameNode라 불리는 중앙노드를 통해 데이터가 저장된 노드를 찾아간다. 

### mapreduce
1. 입력파일을 레코드로 쪼갠다
2. 각 레코드를 key-value로 추출한다. (map)
3. key를 기준으로 정렬한다.
4. 동일 key를 가진 레코드를 병합한다. (reduce)

### 맵리듀스의 분산 실행
- map 태스크에서 실행파일(코드)를 분산 장비로 복사하여 실행한다.
- reduce 태스크에서 같은 키를 가진 데이터들은 같은 장비에서 실행되도록 보장된다.
1. mapper를 통과한 키-값레코드는 정렬되어야하므로, 각 mapper 장비에서 정렬하고, 키의 해시값으로 파티셔닝하여 mapper 장비에 저장한다.
2. 각 reducer는 mapper로부터 완료 이벤트를 받아서, mapper로부터 정렬된 파일을 읽어온다 (= shuffle (섞는다는 의미가 아님))
3. reducer는 다른 mapper로부터 완료이벤트를 받아올때에도, reducer에서는 들고있는 데이터와 병합하여 정렬된 상태를 유지한다.
4. reducer가 작업을 출력파일로 저장하여 마무리한다.

### mapreduce의 workflow
- mapReduce가 한번 더 필요한경우 : (1) accesslog로부터 url별 조회수 추출 -> (2) 인기있는 url 추출 (정렬이 한번더 필요함)
- 하둡에서는 mapReduce의 연결을 지원하지않으므로, 아웃풋 파일을 인풋파일로 사용하여 workflow를 구현한다.
- fig, flumeJava 에서 지원되기도함.

## reduce side join, grouping
- 다른 db에있는 경우 join에 비용이 많이드므로, reduce에 할당된 해시키를 기준으로 조인대상 테이블을 복제하여 reducer에서 join 한다.
- 정렬병합 조인
    - map에서 정렬, reducer에서 보조정렬(key값 별로 join 대상 데이터와 key-value로 온 데이터를 함께 정렬한다.)
    - key당 join대상 레코드 조회를 추가로 할 필요가 없고, 정렬된 상태이므로 처리가 빠를것.
- mapReduce는 내부 네트워크와 애플리케이션 레벨을 분리하여, 네트워크이슈가 발생해도 애플리케이션에 영향없이 재시도한다.

### 그룹화
- map의 key값을 그루핑 키로 하여 처리할 수 있다.
- hot key
    - Pig : 핫키인경우, 해시값으로 reducer에 분배하지않고, 랜덤으로 분배한다. 키가 join할 대상은 모든 reducer에 복제된다.
        - 두번째 mapreduce 작업에서 reducer별 결과를 병합한다.
    - Hive : 핫키를 테이블메타데이터에 저장하고, map side join을 한다.

## Map Side Join
- reduce side join은 입력결과의 포맷이 자유롭다. 하지만 reduce로 갈때 정렬, 조인하는데 비용이 많이든다.
- map side join은 입력 결과의 포맷이 가정되어야한다. 
- 조인 대상 데이터가 매우 작아 mapper 메모리에 올릴수있는경우 적합하다

### broadcast hash join 
- 작은 조인대상 테이블이 모든 입력 파티션으로 broadcast된다는 의미

### partition hash join
- 조인할 두 입력데이터를 같은해시키로 파티셔닝하여, 조인대상 레코드가 모두 같은 mapper 파티션에 위치하는것.

### 맵사이드 병합 조인
- 동일 하게 키로 파티셔닝되어있는 상태라면, 정렬도 가능, 정렬 후 순서대로 읽어 join 하는 방식.
- 맵사이드 병합 정렬이 가능하다는것은, 선행 map reduce에서 정렬과 파티셔닝을 해놓았다는것, 그럼에도 join을 매퍼에서 하는것은 출력값의 차이가 있으므로.
- reduce side join은 조인키로 정렬되어 출력됨.
- map side join은 입력의 파일블록마다 정렬되어 출력됨. 

## 일괄처리 워크플로우의 출력

### 검색 색인 구축
- mapper에서 파티셔닝하고, reducer에서 인덱스 생성하여 저장.

### 일괄처리의 출력으로 키-값을 저장
- 외부 DB로 처리는 네트워크비용, 전체 성공시에만 출력이 생성되는 구조에서는 비효율적이다.
- 대부분 mapReduce 작업 내에서 자체 db에 key-value 형식으로 저장한다.
- 일괄처리 출력에 관한 철학
    - 인적내결함성 : 입력파일이 유지되므로, 버그가 발생해도 패치후 복구가 가능하다.
    - 재시도 : 실패시 결과가 저장되지않으므로, 재시도가 가능.
    - 단일 관심사 : 입력/출력 연결과정은 mapReduce의 관심사가 아니다.
    - unix와 다른점 ? unix는 파일을 부호화하는 부담이 있다 (awk) 하지만, 맵리듀스는 avro같은 구조화된 파일형식을 주로 사용한다.

## 하둡과 분산DB의 비교
- 하둡은 입력데이터에 제한이 없으나, 대규모 분산처리시스템(MPP)는 특정 포맷을 따라야한다.
- 하둡은 데이터 수집속도를 높이고, 데이터 활용방법은 많은 가능성을 열어둘수 있다.  
- 하둡에서 mapReducer를 사용해 MPP로 저장시킬수도 있다.
- MPP에서는 SQL로 간단한 질의를 할수있지만, 복잡한 통합 연산이 필요한경우 코드가 필요하다.
- HDFS와 mapreduce가 있으면 그 위에 sql 실행 엔진 Hive를 올릴 수 있따.
- 하둡 생태계는 Hbase와 MPP같은 Impala 를 포함한다.
>> Hive와 Impala 차이?? 보통 어떤 구조로 사용되는지 알아보자

### 결함을 다루는 방식과 메모리vs 디스크
- MPP
    - 결함 : 장비하나가 죽으면 쿼리 실행이 멈춘다. 새로운 요청 혹은 자동 재시도가 와야 재시도를 할 수 있다. 
    - 메모리 : 가능하면 메모리에 데이터를 많이 올려 조인하거나 쿼리한다.
- 하둡
    - 결함 : 실패한 테스크만 재시도한다.
    - 디스크 : 메모리를 아껴쓴다
    - 배경 : mapreduce는 구글에서 처음 개발했는데, 클라우드 환경에서, 우선순위 높은 작업에 자원을 뺏겨 태스크가 종료될 가능성이 높았으므로, 메모리를 대체로 아껴쓰고, 결함 복원이 잘되게 설계되어있다.

## 맵리듀스를 넘어서

### 중간상태 구체화
- mapreduce workflow에서 출력이 입력으로 넘어갈때 중간 상태가 파일로 저장되어야한다. 
- 이전 결과가 최종 종료되어야지 다음 작업이 진행될 수 있다. unix pipeline은 이전 출력이 바로 다음작업으로 연쇄적으로 넘어간다.
- 임시 데이터도 분산db에 저장되어야하는데, 과잉조치이다.

### 데이터 플로우 엔진
    - spark, tez, flink 이런 엔진들은 전체 워크플로우를 하나의 작업으로 본다.
    - 단일스레드에서 사용자 정의 함수를 레코드마다 반복 실행한다.
- 내결함성
    - 어느 파티션을 할당받았는지, 어떤 operator를 하고있었는지 조회하여 재시도할수있다.
    - spark에서는 RDD 추상화를 사용한다. 
 >> 알아보기
    - operator가 결정적(입력-출력이 동일해야한다) 이어야 완전 내결함성이있다고 볼수있다. 
    - 중간 연산결과가 입력보다 작다면, 하둡처럼 중간상태 구체화가 더 좋을수도있다.
    
### 그래프와 반복처리
- 그래프 DB에서 맵리듀스는 ‘완료될때까지 반복’ 이라는 개념에 적합하지않다. 대개 반복처리로 구현된다.
- 프리글 처리 모델