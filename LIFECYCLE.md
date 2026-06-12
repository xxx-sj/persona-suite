# 개발 라이프사이클 맵 (persona-suite)

> 한 피처가 거치는 전체 흐름 + 각 단계를 어느 도구가 맡나. **각 스킬은 마칠 때 이 표의 *다음 행*을 사용자에게 안내한다 — §단계 전환 프롬프트의 해당 템플릿을 함께 인용한다.**
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
| 8 | 테스트 진행 (단위·통합) | (TDD) · [claude-agents](https://github.com/xxx-sj/claude-agents) test-writer/reviewer | superpowers |
| 8.5 | 프로덕션 리스크 감사 (독립 서브에이전트 판정표) | `auditing-production-risks` | ✅ suite (v1 — 단발 4회 검증/재감사 루프 미검증) |
| 9 | PR·코드리뷰·CI | code-review | superpowers |
| 10 | e2e (staging) | — | 프로젝트 |
| 10.5 | 부하 검증 (조건부 — 요구사항 §5 부하테스트 게이트가 '필요'일 때) | — | 프로젝트 |
| 11 | 배포 (expand-contract·flag) | — | 프로젝트 |
| 12 | 모니터링 + 성공지표 사후 판정 (요구사항 §1 판정 라인 ↔ 실측 대조 → 유지/개선/제거) | — | 프로젝트 (예: 운영 리포트 루틴) |
| 13 | 롤백 / 회고+정리 | — | 프로젝트 |

## suite 경계 (누가 뭘 맡나)
- **persona-suite = 설계(1~5, 페르소나 핑퐁) + 테스트 리스트업(6b) + 프로덕션 리스크 감사(8.5, 판정표).** = superpowers 아닌 라이프사이클 도구 모음.
- **superpowers = 6a 작업분해(writing-plans) · 7~9 구현/테스트/리뷰(TDD·code-review).**
- **프로젝트 인프라 = 10~13 e2e·배포·모니터링·롤백·회고.**
- 외부 도구(superpowers/프로젝트)는 프로필 `handoffs.*`/존재 시 사용, 없으면 산출물에 "다음: X 필요"만 남김(하드 의존 X).

## 현재 → 다음 안내 규칙
각 스킬은 마칠 때 자기 단계의 *다음 행*을 안내하되 **right-size로 건너뛸 수 있다**:
- `defining-requirements` 끝 → DB 닿으면 `designing-data-model`, 아니면 6(작업분해+테스트리스트업).
- `designing-data-model` 끝 → 코드 설계가 *무거우면* `designing-code`, *가벼우면 skip* → 6.
- `designing-code` 끝 → 6.
- 구현·테스트(7~8) 끝 → `auditing-production-risks`(8.5). 재화·외부연동·동시성·큐·마이그레이션 접촉 시 skip 금지, 가벼운 tier 비접촉이면 skip → 9.
- 단계의 무게(비가역·blast radius)에 비례해 깊이 조절. 작은 변경은 1·6·7만 밟아도 됨.
- **단계 전환 = 세션 경계**: 산출물 문서가 동결되면 세션을 끊고(clear), 다음 단계는 문서만 읽고 시작한다 (이전 핑퐁 잔해 = 오염원 + 닻). 이전 대화 없이는 다음 단계가 곤란하다면 그건 문서가 self-contained 하지 않다는 신호 — 문서를 보강한 뒤 끊는다. 단계 *내부*(핑퐁 중)는 끊지 않는다 — 역류가 기능 요건.

## fresh-context 사용 지도 (스텝별)

> **검증 수단 사다리**: oracle(모델이 판정 안 함 — 테스트·프로덕션 지표) > fresh context(같은 모델, 닻 제거) > 같은 컨텍스트 재검토. 각 스텝은 닿는 가장 위를 쓴다 — oracle이 있는 스텝(구현·운영)은 fresh가 주역일 필요 없고, oracle이 없는 스텝(정의·설계)에서만 fresh가 주역.
> **수단 구분**: 서브에이전트 = *검증자*를 깨끗하게(작업 중 닻 없는 시선만 빌려 출력 회수 — 단방향 유리창), clear = *작업자*를 깨끗하게(단계 전환). fresh는 컨텍스트가 새것일 뿐 모델 prior는 공유 — 상세: PRINCIPLES §8(종료 검수)·§10(결정 스팟 반박).

| 스텝 | fresh-context 사용 | md 산출물 |
|---|---|---|
| 1~3 정의 | 핑퐁은 한 세션 유지(역류가 기능). 무거움 tier 정책 확정 직전 §10 반박(서브에이전트, *결정 요지+capacity만* 전달) · 종료 시 §8 검수(서브에이전트, *문서+템플릿만*) | ✅ 요구사항 문서(프로필 output_path) + 최초 1회 persona-profile.md |
| 단계 전환 | **clear** — 다음 단계는 문서만 읽고 시작 (위 "단계 전환 = 세션 경계") | — |
| 4 데이터모델 · 5 코드설계 | 새 세션 자체가 fresh(입력 = 상위 단계 문서). §10·§8 동일 적용 | ✅ 단계별 설계 문서 |
| 6a 작업분해 | 새 세션 + 문서 입력 — 또는 **서브에이전트 위임** (단발 문서 생산이라 가능. 전제: 미결은 plan "Open Questions"로 회수 + 작성 후 실행 미진행. §단계 전환 프롬프트) | ✅ plan 문서 |
| 6b 테스트 리스트 검증 | fresh 불필요 — 판정 기준이 피처 문서(외부 루브릭) | ✗ 판정표는 대화 출력 → plan/스펙에 반영 |
| 7~8 구현·TDD | oracle(테스트)이 판정. 리뷰 에이전트는 read-only 서브에이전트 | ✗ 산출물 = 코드·테스트 |
| 8.5 리스크 감사 | 서브에이전트에 *diff+코드만*(의도·설계 "한계" 미전달) | ✗ 판정표 대화 출력, 항목별 수용/기각 |
| 9 PR·리뷰 | 리뷰어(사람/CI)가 물리적 fresh | PR 본문 |
| 12 사후 판정 | oracle = 프로덕션 지표 | ✗ 판정 한 줄을 요구사항 문서 §1에 *추가* |

**md 규칙**: 설계 단계(1~5·6a)만 문서를 생성한다. 검증 단계(§8 검수·§10 반박·6b·8.5)는 새 문서를 만들지 않고 **기존 문서/코드를 고치는 입력**으로 소비된다 — 진실 문서는 단계당 하나 (source of truth 분산 방지).

## 단계 전환 프롬프트 (스킬 종료 안내용)

> 각 스킬은 마칠 때 다음 행 안내와 함께 해당 템플릿을 그대로 인용한다 — 사용자가 clear 후 새 세션에 복사해 쓰는 용도. `<산출경로>` = 프로필 `conventions.output_path` (예: `features/<도메인>/`), 외부 도구명은 프로필 `handoffs.*` 값으로 치환 (아래 "예:"는 예시일 뿐 하드 의존 아님).

**진입 — 사이클 시작 (새 세션)**
```
/persona-suite:define-requirements <피처 한 줄 설명>
```

**→ 4 데이터모델** (1~3 종료 · DB 닿을 때만 · clear 후)
```
/persona-suite:design-data-model <산출경로>/<피처>.md 기반으로
```

**→ 5 코드설계** (4 종료 · 새 모듈급만 · clear 후)
```
/persona-suite:design-code <산출경로>/<피처>.md 와 <산출경로>/<피처>-datamodel.md 기반으로
```

**→ 6a 작업분해** (마지막 설계 단계 종료 · clear 후 · `handoffs.plan` 도구를 서브에이전트로 위임)
```
<handoffs.plan 도구(예: writing-plans)> 스킬로 구현 계획 작성을 서브에이전트로 돌려줘.
스펙: <이 단계까지의 산출 문서 경로 전부>
탐색: 스펙이 file:line으로 앵커한 파일 + 직접 의존만. 전체 모듈 스캔 금지.
미결 판단은 plan 상단 "Open Questions"에 남기고 임의 결정 금지.
리뷰 루프 5회 초과 시 중단하고 사유 보고. plan 작성 후 실행으로 넘어가지 말 것.
산출: <산출경로>/<피처>-plan.md — 경로 + 태스크 목록 요약만 보고.
```

**→ 6b 테스트 리스트 검증** (6a 직후 · clear 없이 같은 세션 — 위임 덕에 메인이 가볍고 6b는 fresh 불필요)
```
/persona-suite:validate-test-lists <산출경로>/<피처>-plan.md 의 테스트 케이스 목록을 <산출경로>/<피처>.md 기준으로 검증해줘. 판정 결과는 plan에 반영.
```

**→ 7 구현** (6b 종료 · clear 후 새 세션)
```
<산출경로>/<피처>-plan.md 실행해줘. 구현자 서브에이전트는 저렴한 모델로.
```
실행 스킬 이름은 대지 않는다 — plan 헤더의 실행 지시문이 지정한다 (작성 스킬 이름을 잘못 대는 것보다 안전). 구현을 서브에이전트 하나에 통째 위임하지 않는다 — 태스크별 사용자 게이트(요약 리뷰·커밋 게이트·에스컬레이션)가 죽는다.

**→ 8.5 감사 → 9 PR** (구현 종료 · 같은 세션 — 감사는 자체 독립 서브에이전트라 clear 불필요)
```
/persona-suite:audit-production-risks
```
통과(또는 비접촉 skip + 사유 한 줄) 후 "PR 만들어줘".

**clear 지도 (한눈 요약)**
```
진입(1~3) →clear→ 4 →clear→ 5 →clear→ [6a → 6b 같은 세션] →clear→ [7 → 8.5 → 9 같은 세션]
```
작은 변경은 진입·7만 밟아도 된다 (right-size — 위 "현재 → 다음 안내 규칙").