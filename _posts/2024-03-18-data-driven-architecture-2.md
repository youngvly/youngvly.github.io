---
title: "데이터 중심 아키텍처 : 2. 데이터 모델과 질의 언어"
date: 2024-03-18 21:00:00 +0400
categories: study, architecture, db
tags: study, architecture, db
---

# 02. 데이터 모델과 질의 언어
## 관계형 모델과 문서 모델
### NoSQL의 탄생
### 객체 관계형 불일치
- json 표현은 다중테이블 스키마보다 더 나은 Locality를 갖는다. (query하나로 모두 조회가능)
### 다대일과 다대다 관계
- n : 1 ex) 많은 사람들은 한 특정 지역에 산다.
- n : n ex) 많은 사람들은 여러 지역에 분산되어 산다.
### 문서 데이터베이스는 역사를 반복하고있나?
- IMS라는 구 문서 데이터베이스는 현재의 NoSQL처럼 일대다는 잘 동작하지만, 다대다 관계를 중복을 허용할지, 참조를 할지 결정해야했다.
### 네트워크 모델
- 코다실(Conference On DAta SYstem Languages) 에서 표준화하한 모델.
- 네트워크모델에서 레코드간 연결은 외래키보다는 프로그래밍 언어의 포인터와 비슷하며, 최상위 레코드에서부터 연속된 경로를 따르는 '접근경로' 방식을 사용한다.
- 다대다인경우, 모든 연결된 노드를 추적해야하므로 비효율적.
### 관계형 모델
### 문서 데이터베이스와 관계형모델의 비교
- 다대일과 다대다 표현시 관계형과 문서DB모두 외래키/문서참조 방식으로 참조한다.

## 관계형 DB와 오늘날의 문서 DB
### 어떤 데이터 모델이 애플리케이션 코드를 더 간단하게 할까
- 애플리케이션에서 데이터가 문서와 비슷한 구조.
- 로그같은 분석 애플리케이션에서는 다대다 관계가 필요하지않아 NoSQL이 유리.
- 다대다 관계가 필요한경우, join이 가능한 관계형 DB가 유리.
### 문서 모델에서의 스키마 유연성
- schemaless : 문서 DB, 쓸때도 읽을때도 필드의 존재를 보장하지않는다. => 오해의 소지가있어 아래 두개로 나눔.
- schema on write : 관계형DB처럼 쓰여진 모든 데이터가 명시적인 스키마를 따르고있음을 보장한다.
- schema on read : 데이터구조는 암묵적이고 읽을때만 해석된다. 
### 질의를 위한 데이터 지역성
- 전체 문서를 자주 조회해야하는경우, 저장소 지역성을 활용하면 성능 이점이 있다. 
- 다만 문서DB는 효율적인 조회/쓰기를 위해 문서의 크기를 작게 유지해야하는 권장사항이 있다.
- 오라클은 multi-table index cluster table을 제공한다.
### 문서 DB와 관계형 DB의 통합
- mysql 8.0은 둘다가능

## 데이터를 위한 질의 언어
- 명령형 코드 : 그냥 코딩.
- SQL같은 선언형 질의언어에서는, 방법이 아닌 결과와 조건만 선언하면 된다. 방법은 시스템의 질의 최적화기가 할 일.
- 선언형 언어는 순서를 지정하는 명령형보다 병렬 처리에 이점이있다.
### 웹에서의 선언형 질의
- 선언형=css선택자+html / 명령형=js에서 그려주는것 > 명령형은 코드가 복잡하고, 호환성이 좋지않다.
### MapReduce 질의
- 대상을 지정하는 map, 결과를 합치는 reduce 함수를 지정하는것,
- MongoDB의 명령형 질의 mapReduce
```js
  db.collection_name.mapReduce(
    function map() {
      // 입력으로 전달되는 데이터만 사용되어야한다.
      emit(this.timestamp.getMonth());
    },
    function reduce(key,values) {
      return Array.sum(values);
    },
    {
        query : { fieldA : "mustSame"},
        out : "summary" // result collection name
    }
  )
```
- MongoDB의 선언형 질의 aggregation pipeline , 명령형 질의에 비해 질의 최적화기가 질의 성능을 높일 수 있다. 
```js
  db.collection_name.aggregate([
    { $match: { fieldA : "mustSame"}},
    { $group : {
        _id : {
          month : {$month : "$timestamp"}
        },
        "summary" : { $sum : "$number"}
    }}
  ])
```
- NoSQL(Not only Sql) 인 MongoDB도 질의 최적화기가 동작할 수 있는 SQL을 재발견하고있다.
## 그래프형 데이터 모델
- 다대다 관계가 매우 일반적이면, 연결이 복잡해지므로 그래프형 데이터모델이 적합.
  - 소셜 그래프 : 사람(node) + 관계(edge)
  - 웹 그래프 : 웹페이지(Node) + 링크(edge)
  - ex) 네비게이션, PageRank
### 속성 그래프 모델
- Neo4j, titan, infiniteGraph
- node (vertices)
  - 고유한 식별자
  - outgoing/ingoing 간선 집합
  - 속성 컬렉션 (key-value)
- edges
  - 고유한 식별자
  - 시작/끝 노드
  - label
  - 속성 컬렉션 (key-value)
### Cypher 질의 언어
- 속성 그래프를 위한 선언형 쿼리, Neo4j 에서 사용.
```sql
CREATE
  (USA:Location {name:'US',type :'country'})
  (Lucy) -[:BORN_IN]-> (USA)

// 미국으로 온사람
MATCH
  (person) -[:BORN_IN]-> () -[:WITHIN*0..] -> (USA:Location {name:'US'})
RETURN person.name
```
### SQL의 그래프 질의
- SQL로 구현할 수 는 있지만, 그래프 질의언어보다 훨씬 복잡하다. p.56
### 트리플 저장소와 Sparql
- 속성 그래프 모델과 비슷하지만, 모든 정보를 three-part statements 형식으로 저장한다.
```sql
@prefix : <urn:example:>.
_:lucy  a     :Person   # 서술어가 간선이면 목적어는 정점
_:lucy  :name "lucy".   # 서술어가 속성이면 목적어는 문자열
```
### 시멘틱 웹
- 모든 웹사이트가 일관된 형식으로 데이터를 게시하여 'web of data'에 자동으로 결합되도록 하자는 것. 2000년대 초반 과대평가된 이후 사라졌다.
### RDF 데이터 모델
uri로 표현된 xml형식
### 스파클 질의언어
RDF 데이터 모델을 사용한 트리플 저장소의 질의 언어.

### 선언형 질의 세가지
- Cypher
- Sparql
- Datalog

> 이 서비스의 DB구조는 관계형 모델이지만, 정형화를 굳이 하지않는 스키마있는 문서형모델?
> - 불필요한 Join보다 데이터 중복이 쿼리에 효율적이어서,?