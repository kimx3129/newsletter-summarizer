# Newsletter Config

## Topics (priority order)
1. AI/LLM 연구 및 모델 — 새 모델 출시, 논문, LLM 기술 동향 (OpenAI, Anthropic, Google, Meta 등)
2. 빅테크/산업 뉴스 — 빅테크 기업 동향, 인수합병, 규제 이슈
3. 개발자 도구/프레임워크 — 새로운 개발 도구, 라이브러리, 프레임워크 출시/업데이트

## Target output
- 최종 요약 기사 수: 5~10개/일
- 수집 대상 기간: 최근 24~48시간 이내 발행된 기사
- 중복 판정 시 최근 3~5일치 history/ 아카이브와 비교하여 이미 보낸 스토리는 제외 (단, 큰 업데이트가 있으면 예외적으로 재포함 가능)

## Ranking criteria
- 관심 주제와의 직접적 관련성
- 새로움/중요도 (단순 반복 보도, 광고성 콘텐츠 지양)
- 출처 신뢰도 (1차 소스 > 애그리게이터 재게시)

## Delivery
- 채널: Slack (Incoming Webhook)
- Webhook URL은 절대 이 저장소에 커밋하지 않음 — 환경변수 `SLACK_WEBHOOK_URL` 로만 참조
