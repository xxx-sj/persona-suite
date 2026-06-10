# [예시] 카드 재평가 — 데이터 모델 설계 (회차/attempt)

> `designing-data-model` 의 **실전 산출물을 익명화·제네릭화**한 워크드 예시 ([output-template.md](../skills/designing-data-model/output-template.md) 양식). 입력은 [defining-requirements 예시](./defining-requirements-example.md)와 같은 가공 피처 — 두 예시가 한 피처의 연속 산출물이다.
> ⚠️ 도메인·수치·식별자는 가공된 것 — **양식과 깊이만** 참고. output-template.md 가 바뀌면 이 예시도 동기화한다.
> 4번의 🔧 버그 수정은 **AS-IS 조사(절차 0)가 기존 코드의 실제 버그를 잡아낸 장면** — "수정이면 현행 스키마·쿼리를 직접 읽어야" 하는 이유의 실례.

## 1. 입력 / 범위
- 요구사항: [카드 재평가 요구사항 문서](./defining-requirements-example.md)
- 건드리는 것: `Card`(필드 추가) · `CardEvaluation`(필드+인덱스) · 신규 `CardEvaluationAttempt`(audit)

## 2. 데이터 모델 `[REV]`
- `Card.evaluationAttempt: number` (default 1) — 현재 회차. 재평가 시작 시 +1.
- `CardEvaluation.attempt: number` — 이 평가가 속한 회차. `createEvaluationHistory`가 현재 회차 태깅.
- **신규 `CardEvaluationAttempt`** (D-a: 전용 컬렉션) — 회차당 1 doc:
  - `cardId · attempt · startedAt · prevRating · resultRating · status('reevaluating'|'completed') · consumedPoint(3)`
  - = [OPS] admin 조회 + 감사 소스. *무제한 재평가라 Card embed 배열(무한성장) 대신 컬렉션.*
- lifecycle: 1회차(초기) → 재평가 시작 시 `evaluationAttempt++` + CardEvaluationAttempt 생성(reevaluating) → 완료 시 completed + resultRating.

## 3. 인덱스 `[DBA]`
- `CardEvaluation` 추가:
  - `{ cardId:1, attempt:1, rating:-1 }` — getEvaluations 회차 스코핑 top-N (ESR: cardId·attempt equality, rating sort)
  - `{ cardId:1, attempt:1, evaluatorId:1 }` — hasUserEvaluated 회차 스코핑
- `CardEvaluationAttempt`:
  - `{ cardId:1, attempt:1 }` unique — 회차 식별
  - `{ cardId:1, startedAt:-1 }` — CS 최근 재평가 조회
- 악성 필터(`findEvaluatorsWithLowScoreRatio`)는 기존 인덱스로 **회차 무관 유지** (평가자 전체 이력).

## 4. 정합성 / 이력 `[DATA]` `[SEC]`
- **startReevaluation 트랜잭션(원자)**: `consumePoints` + pointHistory(`CARD_REEVALUATION`) + Card(`evaluationAttempt++`, `status='reevaluating'`, evaluationCount/totalRating/evaluatedByUserIds reset) + CardEvaluationAttempt insert. **조건부 원자 업데이트**(`status:'approved'`→`'reevaluating'`)로 동시/멱등.
- **completeEvaluation**: `getEvaluations`/`hasUserEvaluated`에 `attempt: 현재` 필터. 완료 시 `max(기존, calRating)` + CardEvaluationAttempt(`resultRating`·`status='completed'`).
- 🔧 **버그 수정**: `hasUserEvaluatedCard`의 `exists({ _id: cardId })` → `{ cardId: cardId, attempt }` (AS-IS 조사에서 발견 — `_id`로 조회해 거의 항상 false였음).
- 이력 보존: `CardEvaluation.attempt`로 회차별 평가 보존(파괴적 reset 아님). Card의 *카운트만* reset.

## 5. 마이그레이션 `[DATA]` (D-b: 백필)
- **expand**: `evaluationAttempt`·`attempt` nullable 추가.
- **backfill**: 기존 Card `evaluationAttempt=1`, 기존 CardEvaluation `attempt=1` (`updateMany`, T0 소량) → attempt 필터 쿼리가 기존 데이터에도 일관 동작.
- **contract**: 없음(필드 추가만).
- rollback: 필드 drop (additive라 데이터 손실 없음). 코드도 `attempt ?? 1` 방어.

## 6. 한계 / 트레이드오프 `[SRE]`
- `CardEvaluation`: 무제한 재평가 → 카드당 (회차 × 20) 평가 누적(무한). **T0 소량이라 OK.** scale-trigger: 카드당 회차 수가 커지면(수십+) 오래된 회차 평가 아카이빙.
- `CardEvaluationAttempt`: 회차당 1 doc, 작음.
- 인덱스 write: CardEvaluation insert 시 +2 인덱스 — 평가는 hot write 아니라 OK.

## 7. 핸드오프
- 구현 계획 → `handoffs.plan` 으로 (위 🔧 `hasUserEvaluated` `_id` 버그 수정 포함)
- 모듈/코드 설계 → 무거우면 `designing-code` (이 피처는 기존 모듈 수정 위주라 skip — right-size)
