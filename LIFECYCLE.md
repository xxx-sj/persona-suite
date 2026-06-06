# 개발 라이프사이클 맵 (persona-suite)

> 한 피처가 거치는 전체 흐름 + 각 단계를 어느 도구가 맡나. **각 스킬은 마칠 때 이 표의 *다음 행*을 사용자에게 안내한다.**
> 방법론 상세(①②③ 경계·Saga·expand-contract·시니어 포인트): [`reference/backend-feature-cycle.md`](./reference/backend-feature-cycle.md)

| # | 단계 | 맡는 도구 | 구분 |
|---|---|---|---|
| 1 | 요구사항 정의 (왜·누구·성공·정책) | `defining-requirements` ① | ✅ suite |
| 2 | 기능 정의 (밖에서 본 동작·예외) | `defining-requirements` ② | ✅ suite |
| 3 | 기술 설계(고수준) (영향·API·NFR) | `defining-requirements` ③ | ✅ suite |
| 4 | 상세 데이터 모델 (스키마·인덱스·마이그) | `designing-data-model` | ✅ suite |
| 5 | 코드 설계 (모듈·구조) | `designing-code` | ✅ suite (예정) |
| 6 | 작업 분해 + 테스트 리스트업 | `handoffs.plan` + `handoffs.test_list` | 핸드오프 |
| 7 | 구현 | TDD (test-driven-development) | 핸드오프 |
| 8 | 테스트 진행 (단위·통합) | (TDD) | 핸드오프 |
| 9 | PR·코드리뷰·CI | code-review | 핸드오프 |
| 10 | e2e (staging) | — | 프로젝트 |
| 11 | 배포 (expand-contract·flag) | — | 프로젝트 |
| 12 | 모니터링 | — | 프로젝트 |
| 13 | 롤백 / 회고+정리 | — | 프로젝트 |

## suite 경계
- **persona-suite = 상류 설계(1~5)** = 페르소나 핑퐁이 본체.
- **6 이후 = 핸드오프(프로필 `handoffs.*`) / 프로젝트 도구.** 외부 도구(superpowers writing-plans·TDD·code-review 등)는 *있으면 쓰고*, 프로필에 없거나 미설치면 산출 문서에 "다음: X 필요"만 남기고 종료(하드 의존 X).

## 현재 → 다음 안내 규칙
각 스킬은 마칠 때 자기 단계의 *다음 행*을 안내하되 **right-size로 건너뛸 수 있다**:
- `defining-requirements` 끝 → DB 닿으면 `designing-data-model`, 아니면 6(작업분해+테스트리스트업).
- `designing-data-model` 끝 → 코드 설계가 *무거우면* `designing-code`, *가벼우면 skip* → 6.
- `designing-code` 끝 → 6.
- 단계의 무게(비가역·blast radius)에 비례해 깊이 조절. 작은 변경은 1·6·7만 밟아도 됨.