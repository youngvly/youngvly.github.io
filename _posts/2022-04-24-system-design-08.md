# URL 단축기 설계
1. 문제 이해 및 설계 범위 확정.
 - 요구사항
  - 트래픽 확인 > 매일 1억개 생성 >  1160/s
  - url redirection
  - 생성만 지원. 수정삭제 불가.
2. 개략적 설계안 제시 및 동의 구하기.
 - API 엔드포인트
   - 단축용 POST API
   - redirect용 원본 URL을 반환할 GET API
  - URL redirection
3. 