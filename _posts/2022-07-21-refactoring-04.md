---
layout: post
title: "Refactoring 04 테스트 구축하기"
date: 2022-07-21 23:51
categories: Refactoring
tags: cleanCode, study
---
# 04 테스트 구축하기
### 자가 테스트 코드의 가치
- 단위테스트 : 코드의 작은 영역만을 대상으로 빠르게 실행되도록 설계된 테스트
1. 모든 테스트를 완전히 자동화하고 그 결과까지 스스로 검사하게 만들자.
2. 테스트를 자주 수행하는 습관도 버그를 찾는 강력한 도구가 된다.
3. 테스트 스위트는 강력한 버그 검출 도구로, 버그를 찾는데 걸리는 시간을 대폭 줄여준다.
4. TDD
   1. 테스트-코딩-리팩터링
5. 실패한 테스트가 하나라도 있으면 리팩터링하면 안된다.
### 테스트 추가하기
- 테스트는 위험요인을 중심으로 작성해야한다.
- 의미없는 테스트는 리팩터링 비용만 높일뿐.
### 테스트 픽스처 수정하기
- 테스트당 한개만 검증하자.
- given-when-then (조건-발생-결과)
### 경계조건 검사하기
- collection이 비었을때
- 파라미터가 음수일때 등
- 문제가 생길 가능성이 있는 경계조건을 생각해보고 그 부분을 집중적으로 테스트하자.
### 끝나지않은 여정