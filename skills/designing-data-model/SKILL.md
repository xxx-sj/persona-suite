---
name: designing-data-model
description: 요구사항이 정해진 피처의 상세 데이터 모델(스키마·인덱스·마이그레이션·정합성)을 5개 DB 시니어 페르소나(모델링·DBA·데이터·SRE·보안)의 티키타카로 설계한다. defining-requirements 다음 단계. 키워드: 스키마 설계, 데이터 모델, 인덱스 설계, ESR, 마이그레이션, 컬렉션 설계, 정규화, 역정규화, 데이터 모델링.
---

# designing-data-model

요구사항 문서(보통 `defining-requirements` 산출)를 입력으로, 상세 데이터 모델을 페르소나 핑퐁으로 설계한다.
(suite: defining-requirements → **designing-data-model** → designing-code → 구현 계획)

## 공유 원칙 로드 (suite 공통)
플러그인 root [`PRINCIPLES.md`](../../PRINCIPLES.md) 를 읽어 적용한다 — 피드백 모드(결정만 사용자)·개념 본질 도출·right-size·챕터 자기점검(시작 재선언+트리거 점검)·specifics 가정 금지·용어·결정 번복 절차(문서 먼저)·결정 스팟 반박(무거움 tier §10). 이 스킬 변형: **결정 = 모델·인덱스 선택**, right-size = **인덱스 남발·조기 샤딩 금지**.

## 시작 전: 입력 + 프로필
1. 입력 = **요구사항 문서**(features/ 등). 없으면 "어느 피처? 요구사항부터" → `defining-requirements` 권유.
2. 프로필 `<repo>/.claude/persona-profile.md` 로드 (capacity·crosscutting·handoffs). 없으면 객관 사실만 부트스트랩(defining-requirements 규칙 동일).
3. 프로필 `handoffs.db_design`에 **프로젝트 자체 DB 자산**(기존 스키마 문서/페르소나 등)이 있으면 → **선택 참조**(그 맥락·정책을 압박에 주입, 결과를 그 형식으로도 정리). 없으면 자체 페르소나로 진행 (portable).

## 페르소나 로드
이 디렉토리 `personas.md`의 5종(`[REV] [DBA] [DATA] [SRE] [SEC]`) 로드. `[REV]`(모델링) 메인, 트리거로 나머지 호출. 사용자가 개별 호명("지금 [DBA]만") 가능.

## 진행 절차
```
0. 입력 로드 + right-size + **수정이면 현행 스키마·쿼리·인덱스 AS-IS 조사** (질문 한 줄이 아닌 일급 step — 새 필드가 들어갈 기존 schema + 그걸 읽는 쿼리/인덱스 메서드를 *직접 읽어야* [REV]/[DBA] 초안이 정확. 작은 변경이면 [REV][DBA] 빠른 패스)
1. [REV]  데이터 모델 — 새 엔티티/필드: 별도 collection vs embed? 관계·lifecycle·대안 비교
2. [DBA]  인덱스 — 쿼리 패턴 → 인덱스, ESR(Equality→Sort→Range) 순·prefix·write 비용·partial·covered
3. [DATA] (denorm/이력/통계 트리거) — source of truth·drift 감지·backfill·이력 collection·마이그 default
4. [SRE]  (hot path 트리거) — 부하·첫 병목·캐시·transaction retry(WriteConflict)·graceful degrade
5. [SEC]  (PII/금전/감사 트리거) — PII·암호화·소프트삭제 충분?·로그 마스킹·보관/감사
6. 마이그레이션 전략 — expand→백필→contract(무중단)·default·롤백
7. [REV] 종료 자기 점검 — 빠진 거 훑기
8. 산출 → 데이터 모델 설계 doc (output-template)
9. 종료 독립 검수 (공유 원칙 §8) — 완성 문서 독립 서브에이전트 1회 → handoffs.plan 핸드오프
```

## 출력
`output-template.md` 양식. 위치 = 프로필 `conventions`(요구사항 문서 옆 또는 `doc_dir`). 프로젝트에 기존 스키마 문서 시스템(handoffs.db_design) 있으면 그 형식으로도 정리/참조.

## 핸드오프
> 전체 단계·다음 안내: 플러그인 root [`LIFECYCLE.md`](../../LIFECYCLE.md).
- 구현 계획 → `handoffs.plan` (예: writing-plans, superpowers면 OK)
- 코드 설계 → suite `designing-code` (있으면)

## 금지
- 요구사항 없이(가상 스키마) 설계.
- 모델·인덱스 *선택*을 페르소나가 대신 결정 (제안·압박까지, 결정은 사용자).
- capacity 무시한 과설계 (인덱스 남발·조기 샤딩/파티셔닝).