# [예시] 카드 재평가 — approved 카드 유료 재평가 (max 갱신)

> `defining-requirements` 의 **실전 산출물을 익명화·제네릭화**한 워크드 예시 ([output-template.md](../skills/defining-requirements/output-template.md) 양식). "완성된 요구사항 문서가 어느 정도 깊이·밀도인가"의 캘리브레이션 기준으로 쓴다.
> ⚠️ 도메인·수치·코드 위치(file:line)는 가공된 것 — **양식과 깊이만** 참고. output-template.md 가 바뀌면 이 예시도 동기화한다.
>
> 맥락: 콘텐츠 카드 플랫폼. 사용자가 카드를 만들면 평가단 20명이 심사해 점수(rating)·등급이 붙고, 시작 시 유료 재화(포인트) 3개를 소비한다. 이 피처는 "이미 심사 끝난(approved) 카드를 돈 내고 다시 심사받아 점수를 올릴 기회"를 추가한다.

## 1. 개요 (Context)
- **문제(Why)**: 포인트는 유료 재화인데 **소모처(sink)가 부족**. 포인트 소비를 늘려 매출을 촉진해야 함.
- **해결책**: 이미 평가 완료(approved)된 카드를 **다시 평가(재평가)** 받게 하고, 시작 시 **3포인트 소비**. 점수는 **떨어지지 않고 오르기만** 함(자랑·컬렉션 동기와 정합).
- **지표**: 재평가발 포인트 소비(=매출). `pointHistory` reason `CARD_REEVALUATION` 태깅으로 측정.

## 2. 확정 정책
- 비용: 재평가 **시작 시 3포인트 선소비**, **환불 없음** (무산돼도 반환 X).
- 점수: 새 회차 완료 시 `rating = max(기존, 재계산)` — 같거나 낮으면 기존 유지.
- 횟수: **무제한**, 쿨다운 없음. (인플레·도박성 **의도적 허용** — downstream에서 rating을 비교·소비하지 않아 변별력 무해.)
- 대상: **approved 인 카드만**.
- 평가자: 회차는 서로 다른 20명. **직전 회차 평가자 재참여 허용**(회차 내 중복만 금지). 악성 평가자 점수 제외 유지.
- 권한: **소유자(만든 사람)만** 재평가 시작.
- 법무/스토어: **미확인 — 감수**. 완화책 = framing "재평가 *서비스* 1회 판매(점수 상승 보장 아님)" + 시작 전 환불불가 고지(프론트). 출시 전 법무 1회 확인 *권장*.

## 3. AS-IS / TO-BE
- **AS-IS** (`cards.service.ts`): `create()`(:499) 3포인트 차감 → `status='review'` → 서로 다른 20명 평가(악성 제외) → 20번째 완료 시 상위19 평균 = `rating`, `status='approved'`+`ratingColor`(:819~827). `approved`가 종착, **재평가 경로 없음**. `deleteCard`(:667)는 approved 아니면 차단.
- **TO-BE**: approved 카드에 `startReevaluation` 진입점 추가. `status='reevaluating'`(신규)로 새 회차 진행(기존 `completeEvaluation` 재사용), 완료 시 `max` 적용 후 `approved` 복귀. 재평가 중 공개 노출 제외(소유자만 봄).

## 4. 요구사항 + 예외 (관찰 가능한 결과)
- **기본 흐름**: approved + 소유자 + 잔액≥3 → 3포인트 차감 + 회차 reset + `status='reevaluating'` + 큐 등록 → 서로 다른 20명 평가(직전 평가자 OK, 악성 제외) → 20명 완료 → `max(기존, 재계산)` 적용 → `approved` 복귀.
- **상태 전이**: `approved → reevaluating →(20명 완료)→ approved`.
- **예외**:

  | 상황 | 관찰 결과 |
  |---|---|
  | 잔액 < 3 | 시작 거부, 차감 0, status·attempt 불변 |
  | 소유자 아님 | 거부, 차감 0 |
  | status가 approved 아님(review/reevaluating/cancelled/banned) | 거부 (전제: approved만 시작) |
  | 동시 2요청(둘 다 approved 읽음) | 원자적으로 1개만 성공(3포인트 1회·attempt+1), 나머지 거부 |
  | 새 회차 완료 & 새>기존 | rating·ratingColor 갱신, approved 복귀 |
  | 새 회차 완료 & 새≤기존 | rating 기존 유지, approved 복귀, 3포인트는 소비됨 |
  | 회차 내 동일 평가자 중복 | 거부 |
  | 재평가 중 삭제 시도 | 거부(approved 아님) |
  | 20명 미충족 | reevaluating 유지(미완 대기), **복구 없음** |
  | 재평가 중 ban | 큐 제외, status='banned' override, 환불 없음 |
- **노출**: reevaluating 중 공개 목록·조회 제외, 소유자 조회엔 `reevaluating` 노출.

## 5. 비기능 (NFR)
- **재화 정합성**: 차감+이력+회차reset+status+큐 = 한 트랜잭션, 실패 시 전부 롤백. (`create` 패턴 재사용)
- **동시성/멱등**: 단일 문서 조건부 원자 업데이트(`status:'approved'`→`'reevaluating'`)로 더블클릭 1개만 승리.
- **성능/throughput** ⚠️: 무제한 재평가 → 평가 큐 적체 가능(T0). 관측(미완 적체) + scale-trigger(적체 심하면 횟수 제한).
- **관측성**: 재평가 매출·횟수 + 미완 적체 운영 모니터링.

## 6. AC / 검증 기준
AC 1~13 (위 예외표 + 노출 = 검증 가능한 최종 상태). 핵심:
- 시작 정상: 포인트 -3·`pointHistory` 1건·`reevaluating`·`attempt+1`·회차 카운트 reset.
- 잔액부족/소유자아님/비approved: 거부 + 차감 0 + 불변.
- 동시 2요청: 최종 -3만·attempt+1·1건 (관찰 상태로 검증, mock 단언 X).
- max: 새>기존 갱신 / 새≤기존 유지, 둘 다 approved 복귀 + 3포인트 소비.
- 악성 제외·회차 내 중복 거부·직전 평가자 허용·재평가 중 삭제 거부·미충족 시 미완 유지.
- **→ 테스트 리스트는 `handoffs.test_list` 타깃으로 검증.**

## 7. 운영 흐름
- 고지: 시작 전 환불불가 + "재평가 서비스 1회" framing (**프론트** 담당).
- admin 조회: 기존 유저/거래 조회 **확장** — 재평가 시작 이력(시각·회차 차수·소비 3포인트) + 결과(이전점수→이후, 올랐/유지) 포함. (전용 화면 불필요)
- 모니터링: 재평가 매출·횟수 / 미완 적체.
- 운영자 액션: 등록=사용자 자동. **수정·취소·강제환불·복구 없음.** CS는 "환불 불가" 안내 + 위 조회로 결과 확인.

## 8. 한계 / 트레이드오프
- **미완 재평가 = 카드 무기한 hidden + 3포인트 소비, 복구 없음** (새 status 선택 + 무제한의 대가, *수용*). throughput 느리면 체감 커짐.
- 점수 인플레·도박성 허용 (downstream 무해라 감수).
- 담합(지인 동원 반복 5점) 여지 — 기존 악성필터+self-eval로 충분하다 *감수*.
- 법무/스토어 미확인 (framing+고지로 완화, 출시 전 확인 권장).

## 9. 핸드오프 메모
- **db_design** (✅ `designing-data-model` 확정 → 데이터 모델 설계 문서로 산출): 회차(attempt) 스키마(`Card.evaluationAttempt` + `CardEvaluation.attempt` + `CardEvaluationAttempt` + 인덱스), audit 필드(`startedAt·prevRating·resultRating`), 백필 마이그.
- **test_list**: AC 1~13 기반 테스트 리스트 검증 (특히 #5 동시성·#10 악성 제외·#6/7 max 경계).
- **plan**: 구현 계획.
- **알림 (확정)**: 재평가 완료 시 **별도 이벤트 `CARD_REEVALUATION_COMPLETED`** 발행 — 결과(올랐/유지) 구분 ❌, "재평가 완료" *재유입* 알림. `completeEvaluation`이 2회차 이상(재평가)이면 기존 `CARD_EVALUATION_COMPLETED` 대신 이걸 발행. (발행은 테스트 stub — 비AC)
