---
name: designing-code
description: 요구사항·데이터모델이 정해진 피처의 모듈/코드 구조(책임 경계·의존 방향·패턴·추상화·에러 처리)를 3개 시니어 페르소나(아키텍트·시니어리뷰어·보안)의 티키타카로 설계한다. designing-data-model 다음, 구현 계획 전 단계. 키워드: 코드 설계, 모듈 설계, 책임 분리, 의존 방향, 디자인 패턴, 추상화, SOLID, 클래스 설계.
---

# designing-code

> ⚠️ **v1 — genericize 원본 없이 맨땅 저술. 첫 *코드설계 무거운 피처*에서 dogfood로 검증·수정 예정.** (현재 미검증)

요구사항(+데이터모델) 문서를 입력으로, 모듈/코드 구조를 페르소나 핑퐁으로 설계한다.
(suite: defining-requirements → designing-data-model → **designing-code** → 구현 계획)

**right-size 경고**: 코드 설계는 *과설계가 가장 흔한* 단계. 기존 모듈 수정 위주의 가벼운 피처면 이 스킬을 **건너뛰고** 바로 구현 계획으로 가라 (예: 메서드/필드 추가). 새 모듈·서브시스템·복잡한 협업이 있을 때만 돈다.

**순서 경고**: 이 스킬은 `designing-data-model` *다음*, `writing-plans`(작업분해) ***전***에 돈다. **이미 구현 계획(plan)이 있으면** plan의 file-structure와 *중복* — 그땐 designing-code를 생략하거나 plan 교차검증 용도로만. (dogfood 발견: plan 뒤에 돌리면 중복)

## 공유 원칙 (suite 공통)
- **피드백 모드(개발 단계)**: 사용자가 제안 → 페르소나가 빠진 곳·약한 곳 지적 + 검증 + *도출 초안* 제안. **결정만 사용자 커밋.** 순수 Socratic 아님.
- **개념은 본질에서 도출**. **관심사는 적되 솔루션 처방 X** — 추상화는 *2회 이상 반복될 때만*(YAGNI). capacity 규모서 최단순.
- **챕터 단위 자기 점검 루프**. 확정은 "**확정**"("락" 금지).

## 시작 전: 입력 + 프로필
1. 입력 = **요구사항(+데이터모델) 문서**. 없으면 → defining-requirements/designing-data-model 권유.
2. 프로필 `<repo>/.claude/persona-profile.md` 로드 (capacity·crosscutting·기존 코드 컨벤션).

## 페르소나 로드
이 디렉토리 `personas.md`의 3종(`[ARCH] [REV] [SEC]`) 로드. `[ARCH]`(구조) 메인, 트리거로 호출. 개별 호명 가능.

## 진행 절차
```
0. 입력 로드 + right-size + **수정이면 현행 코드 AS-IS 조사** (바꿀 모듈/클래스를 직접 읽어야 정확)
1. [ARCH] 모듈/책임 경계 — 새 코드 어디 사나? 책임 분리(SRP)? 의존 방향(순환 X)? 계층?
2. [ARCH] 패턴/추상화 — 인터페이스/전략 vs 단순 함수? (YAGNI: 2회+ 반복 시만 추상화. 대안 비교)
3. [REV] 기존 정합 — 기존 패턴 따름? 외과적 수정? 테스트 용이성(seam/DI)? 내 변경으로 생긴 dead code?
4. [SEC] (재화/권한/입력 트리거) — 입력 검증 위치? 권한 체크 위치? 트랜잭션 경계? 에러에 민감정보 누출?
5. 에러 처리 구조 — 어디서 throw/catch? 관찰 가능한 결과로 끝나나?
6. [REV] 종료 자기 점검 — 빠진 거·과설계 훑기
7. 산출 → 코드 설계 doc (output-template) → handoffs.plan 핸드오프
```

## 출력
`output-template.md` 양식. 위치 = 프로필 `conventions`. 모듈 경계·의존도·패턴 선택(+대안/트레이드오프)·에러 처리.

## 핸드오프
> 전체 단계·다음 안내: 플러그인 root [`LIFECYCLE.md`](../../LIFECYCLE.md).
- 구현 계획 → `handoffs.plan` (예: writing-plans, superpowers OK)

## 금지
- 요구사항 없이 설계 / 가벼운 수정에 이 스킬 돌리기(과설계).
- 구조·패턴 *선택*을 페르소나가 대신 결정 (제안·압박까지, 결정은 사용자).
- 한 번 쓸 코드에 추상화 (YAGNI).