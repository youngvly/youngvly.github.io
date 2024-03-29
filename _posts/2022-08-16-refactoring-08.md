---
---
layout: post
title: "Refactoring 08 기능이동"
date: 2022-08-15 23:51
categories: Refactoring study
tags: cleanCode, study
---
# 08 기능이동
## 8.1 함수 옮기기
- 대상함수를 호출하는 함수
- 대상함수가 호출하는 함수
- 대상 함수가 사용하는 데이터 살펴보기
## 8.2 필드 옮기기
- 현재 데이트 구조가 적절치 않음을 깨달으면 곧바로 수정해야한다 -> db까지 영향이 갈수있어 어려운일..
- Account(n) -> AccountType(1) 필드 이동시 , Account별로 해당필드데이터가 다를수있으므로, Account constructor에 assertion을 적용하여 시스템을 잠시 운영해보자. 
## 8.3 문장을 함수로 옮기기
- 반대 : 8.4 문장을 호출한곳으로 옮기기
- 중복코드 제거, 코드라인을 함수에 포함시키는것.
- 대상 라인이 피호출함수의 일부라는것이 확실해야한다.
## 8.4 문장을 호출한곳으로 옮기기
- 반대 : 8.3 문장을 함수로 옮기기
- 작은 변경 : 문장을 호출한 곳으로 옮기기
  1. 옮기지않을 코드만 모은 메서드를 새로운 이름으로 생성한다.
  2. 새로운 함수 + 옮길 문장 인라인
  3. 테스트
  4. 새 메서드 이름 수정
- 호출자와 대상의 경계를 완전 다시 설정할때 : 함수 인라인하기 -> 문장 슬라이드하기 -> 함수 추출하기
## 8.5 인라인 코드를 함수 호출로 바꾸기
- 이미 존재하는 함수와 같은 일을하는 인라인 코드를 발견하면, 호출로 대체하자.
- 이름을 잘 지었다면 함수이름을 넣어도 말이 된다.
- 라이브러리가 제공하는 함수로 대체하면 더 좋다.
- 6.1 함수 추출하기는 없는 함수를 만드는것, 8.5는 존재하는 메서드를 사용하는것.

## 8.6 [문장 슬라이드하기](https://refactoring.com/catalog/slideStatements.html)
- 조건문의 공통 실행코드 빼내기 , 문장 재배치
- 변수를 처음 사용하는곳으로 이동해주면, 함수추출이 편하다.
- 변수를 수정하는 문장을 건너뛰고 문장을 이동할 수 없으므로, 호출되는 메서드가 부수효과가있는지 확인하자.


## 8.7 [반복문 쪼개기](https://refactoring.com/catalog/splitLoop.html)
- 반복문을 분리하면 그 값만 바로 반환할 수 있지만, 여러일을 수행하는 반복문이라면 구조체를 반환하거나 지역변수를 활용해야한다.
- 반복문과 함수 추출하기는 연이어 수행하는일이 많다.
- 리팩터링과 최적화를 구분하자. (성능에는 큰영향이 없다.)
> 반복문 쪼개기의 성능을 알아보자
### 절차
1. 반복문을 복제해 두개로 만든다.
2. 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.
3. 테스트
4. 각 반복문을 함수로 호출할지 고민해본다.
   1. 문장 슬라이드하기 (8.6)
   2. 함수로 추출(6.1)
   3. 파이프라인으로 바꾸기(8.8) , 알고리즘 교체하기(7.9) 까지 적용 가능.


## 8.8 [반복문을 파이프라인으로 바꾸기](https://refactoring.com/catalog/replaceLoopWithPipeline.html)
### 절차
1. 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만든다.
2. 반복문의 첫줄에서 시작해서 각각의 단위 행위를 적절한 파이프라인으로 대체한다. -> 이전 연산을 기초로 연쇄적으로 수행되며 -> 추가할때마다 테스트
3. 반복문의 모든 동작을 대체했다면 반복문 지우기


## 8.9 [죽은코드 제거하기](https://refactoring.com/catalog/removeDeadCode.html)
버전관리시스템을 믿고 제거하자. 어느 리비전에서 삭제했는지를 커밋메시지로 남겨두자.