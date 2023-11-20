---
layout: post
title: "Refactoring 06 기본적인 리팩터링"
date: 2022-07-30 23:51 
categories: Refactoring study
tags: cleanCode, study
---
# 06 기본적인 리팩터링
## 6.1 함수 추출하기
- 반대 : 함수 인라인하기 (6.2)
- 묶는 기준
   1. 코드 길이
   2. 재사용성
   3. 목적과 구현의 분리
- 함수가 짧으면 캐싱하기 더 쉬워서 컴파일러가 최적화하는데 유리할때가 더 많다.
- ex) 지역변수의 값을 변경할때
   1. 매개변수에 값 이 들어간다면, 변수로 추출하자.
   2. 지역변수 초기화와 실사용이 떨어져있다면 초기화 코드를 이동하자. (문장 슬라이드하기 8.6)
   3. 이 지역변수가 추출한 함수 밖에서 사용된다면 return하자.
   ```java
      func calc(invoice){
         let temp = 0
         for () {...}
         return temp 
      }
      
      let outstanding = calc(invoice)
   ```
- 값을 반환할 변수가 여러개라면?
   - 임시 변수를 질의함수로 바꾸거나(7.4)
   - 변수를 쪼개는 식(9.1)
   - 레코드로 묶어서 반한하는것도 가능.

## 6.2 함수 인라인하기
- 반대 : 함수 추출하기 (6.1)
- 함수 본문이 이름만큼 명확한 경우
## 6.3 변수 추출하기
- 반대 : 변수 인라인하기 (6.4)
- 클래스안에서는 getter 로 추출해도 좋다.
- 변수를 불변으로 만들자.
## 6.4 변수 인라인하기
- 반대 : 변수 추출하기 (6.3)
- 변수 명이 원래표현식과 다를바 없다면
## 6.5 함수 선언 바꾸기 = 시그니처 바꾸기
- 함수 이름이 적절하지 않아보이면 즉시 교체하자.
- 주석을 이용해 함수의 목적을 설명하다보면 좋은 이름이 떠오르기도 한다.
- 공개 API 리팩터링시, 새 메서드를 만들고 기존을 deprecated 하여 점차마이그레이션 한다.
- 매개변수 추가
   - js라면 컴파일시에 누락부를 찾을수없기때문에, 함수 시작부에 assertion을 두어 파라미터가 잘 넘어왔는지 확인한다.
## 6.6 변수 캡슐화하기
- getter/setter 로 감싸기
- 데이터 변경 및 사용의 확실한 통로이므로, 로직추가가 쉽다.
- 값 캡슐화하기
- getter가 값의 복제본을 반환하도록 하자.
## 6.7 변수 이름 바꾸기
- 폭 넓게 쓰이는 변수라면 캡슐화하기를 고려한다.
   - 변수를 읽고, 쓰는 용도라면 6.6 캡슐화하기.
- 한번에 수정이 어렵다면, 원본이름을 바꾼 후 복제본을 만들어 점진적으로 바꾸자.
```javascript
const companyName = '어떤값' // new
const CpName  = companyName // old
```  
## 6.8 매개변수 객체 만들기
- 데이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해진다.
- 데이터 구조를 받게하면 매개변수의 수가 줄어든다.
## 6.9 여러 함수를 클래스로 묶기
- (흔히 함수 인수로 전달되는) 공통 데이터를 중심으로 긴밀하게 엮여 작동되는 함수무리를 클래스로 묶자.
- 클래스로 묶을때 장점: 클라이언트가 객체의 핵심 데이터를 변경할 수 있고, 파싱 객체들을 일관되게 관리할 수 있다.
- 원본데이터가 코드안에서 갱신될때는 클래스로 묶는것이 좋다. (6.11과 비교시)
```java
// ASIS
func base(aReading)
func taxableCharge(aReading)
func calculate(aReading)
// TOBE
class Reading{
   base()
   taxableCharge()
   calculate()
}
```  
## 6.10 여러 함수를 변환 함수로 묶기
- 6.9와 차이는 clone되고, 원본데이터를 수정하지않음.
```java
// ASIS
func base(aReading)
func taxableCharge(aReading)
func calculate(aReading)
// TOBE
func gather(argReading){
   const aReading = argReading.deepClone()
   aReading.baseCharge = base(aReading)
   aReading.taxableCharge = taxableCharge(aReading)
   return aReading
```
## 6.11 단계 쪼개기
- 서로다른 두 대상을 한꺼번에 다루는 코드를 별개의 모듈로 나누자.
- ex) 입력이 처리로직에 적합하지않은 형태로 들어오는경우, 가공과 처리를 분리.
``` javascript
// ASIS
const orderData = orderString.split('/')
const productPrice = priceList[orderData[0].split('-')[1])
const orderPrice = parseInt(orderData[1]) * productPrice;
// TOBE
const orderRecord = parseOrder(orderString)
const orderPrice = price(orderRecord,priceList)
``` 

