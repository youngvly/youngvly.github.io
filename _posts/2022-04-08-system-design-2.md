---
layout: post
title: "System design 2 개략적인 규모 추정"
date: 2022-04-08 19:10:00 +0400
categories: Design
tags: design
---
# 2. 개략적인 규모 추정
## 2의 제곱수
제대로된 계산결과를 얻기위해 데이터 볼륨의 단위를 2의 제곱수로 표현하려면 어떻게 되는지 알아야한다.
ASCII문자 1개가 1byte

- 2^10 = 1천 = 1KB
- 2^20 = 1백만 = 1MB
- 2^30 = 10억 = 1GB
- 2^40 = 1조 = 1TB
- 2^50 = 1000조 = 1PB
## 응답지연값
2020
![image](https://user-images.githubusercontent.com/19251378/162389445-8f40834b-1de8-4ee2-97ad-e151cc862296.png)
- [시각지표 참고](https://colin-scott.github.io/personal_website/research/interactive_latency.html)
- 메모리는 빠르지만 디스크는 아직 느리다.
- 단순 압축 알고리즘은 빠르다
- 데이터센터간 패킷이동 소요시간을 무시할수없다.
## 가용성 관련 수치
| 가용률  | 하루 장애 시간 | 주당  | month |    year |
| :------ | -------------- | ----- | ----- | ------: |
| 99%     | 14.4 min       | 1.68h | 7.31h | 3.65day |
| 99.999% | 864ms          | 6.05s | 26.3s | 5.26min |
## 계산 예제 서적 참고

