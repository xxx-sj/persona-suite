# 개발 라이프사이클 맵 (persona-suite)

> 한 피처가 거치는 전체 흐름 + 각 단계를 어느 도구가 맡나. **각 스킬은 마칠 때 이 표의 *다음 행*을 사용자에게 안내한다.**
> 방법론 상세(①②③ 경계·Saga·expand-contract·시니어 포인트): [`reference/backend-feature-cycle.md`](./reference/backend-feature-cycle.md)

| # | 단계 | 맡는 도구 | 구분 |
|---|---|---|---|
| 1 | 요구사항 정의 (왜·누구·성공·정책) | `defining-requirements` ① | ✅ suite |
| 2 | 기능 정의 (밖에서 본 동작·예외) | `defining-requirements` ② | ✅ suite |
| 3 | 기술 설계(고수준) (영향·API·NFR) | `defining-requirements` ③ | ✅ suite |
| 4 | 상세 데이터 모델 (스키마·인덱스·마이그) | `designing-data-model` | ✅ suite |
| 5 | 코드 설계 (모듈·구조) | `designing-code` | ✅ suite (v1 — light 검증/heavy 미검증) |
| 6a | 작업 분해 | `handoffs.plan` (예: writing-plans) | superpowers |
| 6b | 테스트 리스트업 (빠진/불필요/**엣지** 검증) | `validating-test-lists` | ✅ suite |
| 7 | 구현 | test-driven-development | superpowers |
| 8 | 테스트 진행 (단위·통합) | (TDD) | superpowers |
| 9 | PR·코드리뷰·CI | code-review | superpowers |
| 10 | e2e (staging) | — | 프로젝트 |
| 11 | 배포 (expand-contract·flag) | — | 프로젝트 |
| 12 | 모니터링 | — | 프로젝트 |
| 13 | 롤백 / 회고+정리 | — | 프로젝트 |

## suite 경계 (누가 뭘 맡나)
- **persona-suite = 설계(1~5, 페르소나 핑퐁) + 테스트 리스트업(6b, `validating-test-lists` 검증).** = superpowers 아닌 라이프사이클 도구 모음.
- **superpowers = 6a 작업분해(writing-plans) · 7~9 구현/테스트/리뷰(TDD·code-review).**
- **프로젝트 인프라 = 10~13 e2e·배포·모니터링·롤백·회고.**
- 외부 도구(superpowers/프로젝트)는 프로필 `handoffs.*`/존재 시 사용, 없으면 산출물에 "다음: X 필요"만 남김(하드 의존 X).

## 현재 → 다음 안내 규칙
각 스킬은 마칠 때 자기 단계의 *다음 행*을 안내하되 **right-size로 건너뛸 수 있다**:
- `defining-requirements` 끝 → DB 닿으면 `designing-data-model`, 아니면 6(작업분해+테스트리스트업).
- `designing-data-model` 끝 → 코드 설계가 *무거우면* `designing-code`, *가벼우면 skip* → 6.
- `designing-code` 끝 → 6.
- 단계의 무게(비가역·blast radius)에 비례해 깊이 조절. 작은 변경은 1·6·7만 밟아도 됨.