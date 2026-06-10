---
description: 작성하려는 테스트 케이스 목록을 피처 정의 기준으로 검증한다 — 빠진/불필요/엣지 케이스 (라이프사이클 6b).
---

# 테스트 리스트업 검증 (단계 6b)

`persona-suite:validating-test-lists` 스킬을 실행한다. 대상: "$ARGUMENTS"

**호출 범위**: 이 커맨드는 *테스트 리스트 검증(단계 6b)만* 실행한다 — 피처 정의 기준으로 KEEP/MODIFY/DELETE/ADD(엣지 포함) 판정. 입력으로 피처 정의(요구사항 문서) + (있으면) 검증할 테스트 리스트가 필요하다.
**다음 단계는 자동 실행하지 않는다** — 검증 후 구현(LIFECYCLE 7단계, 예: test-driven-development)으로 사용자가 직접 진행.