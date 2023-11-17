---
title: "SoftwareEngineering at google : 15 폐기"
date: 2023-05-24 19:10:00 +0400
categories: softwareEngineering
tags: SoftwareEngineering
---

# 15. 폐기
- 코드는 자산이 아니라 부채다
- 구형과 신형 시스템을 동시에 활용하는것은 하위호환성 유지 비용이 결국 신식 시스템 개선 작업 지연에 영향을 끼친다.
## 폐기가 어려운이유
- 하이럼의 법칙, 의도치않은 기능을 클라이언트에서 사용하고있을 수 있어 클라이언트가 떨어져나갈수도 있음.
- 반대세력, 구글은 코드 레포에서 과거이력까지 검색할 수 있게 하여 거부감을 줄이려고함.
- 폐기의 필요성 어필, 7장 참고.
- 점진적으로 진행하는것이 부담도적고, 이용자 불편도 줄임.
### 설계안계에서의 폐기
- 엔지니어링 팀에게 권장하는 고려사항
  - 내 제품의 고객이 잠재적인 대체품으로 이주하기 얼마나 쉬울까
  - 내 시스템을 한부분씩 점진적으로 교체하려면 어떻게 해야할까
- 회사에서 기대수명동안 제대로 지원하지 못할것같은 프로젝트는 시작하지 말라.

## 폐기유형
### 권고폐기 (희망 폐기)
- 강제성 없음
- 우선순위가 높지 않음.
- 기한 없음.
- 사용자들을 원하는 방향으로 움직이게 찔러보는 정도, 앱 업데이트?
### 강제 폐기
- 구 시스템의 지원 종료일을 못박는 형태.
- 전문팀을 꾸려 적극적으로 지원해야한다. 일정대로 집행할 수 있는 권한이 있어야하고, 구시스템 문제에 책임을 묻지않아야 한다. 
### 폐기 경고
- 실행가능성 : 구시스템을 사용하는 클라이언트에게 대체제를 알려주어야 한다.
- 적시성 : 되도록 빨리 코드를 작성할때 알려주는게 좋다.
## 폐기 프로세스 관리
1. 프로세스 소유자를 지정하고 폐기까지 책임져야한다.
   - 소유자 관리가 안된다면 폐기 전문가 배정
2. milestone : 폐기할때도 점진적 성취를 칭찬할수있어야한다.
### 폐기 도구
1. 발견 : 누가 어떻게 이용하고있는지 (CodeSearch, Kythe)
2. 마이그레이션
3. 퇴행 방지 : 폐기 대상을 이용하는것, 경고해주어야 한다. (Tricorder)
