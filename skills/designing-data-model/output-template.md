# 데이터 모델 설계 출력 양식

`designing-data-model` 산출물. 위치 = 프로필 `conventions`(요구사항 문서 옆 또는 `doc_dir`). 해당 없으면 "해당 없음" 명시(빈칸 방치 금지). 완성 깊이의 기준은 [`examples/designing-data-model-example.md`](../../examples/designing-data-model-example.md) 참고.

---

# <피처> — 데이터 모델 설계

## 1. 입력 / 범위
- 요구사항 출처: <features/.../<피처>.md 링크>
- 건드리는 엔티티/컬렉션: <목록>

## 2. 데이터 모델
- 신규/변경 엔티티: <collection vs embed, 필드(타입·nullable 의미)·관계·owner>
- lifecycle: <생성 → 변경 → 종결>
- 대안 비교: <검토한 대안 + 왜 안 골랐나>

## 3. 인덱스
| 인덱스 | 받는 쿼리 | ESR 검토 | 비고(partial/covered/write비용) |
|---|---|---|---|
| <{a:1,b:-1}> | <쿼리> | <E/S/R 순 맞나> | <...> |

## 4. 정합성 / 이력
- 트랜잭션 경계: <어디서 어디까지 원자적>
- denormalized 필드 source of truth + drift 감지: <...>
- 이력 필요 여부: <append history collection vs 현재값만>

## 5. 마이그레이션 전략
- expand → 백필 → contract 순서: <...>
- default(기존 document 처리): <null/0/미존재>
- 롤백: <되돌리는 법>

## 6. 보안 (해당 시)
- PII / 암호화 / 소프트삭제 / 로그 마스킹 / 보관·감사: <...>

## 7. 한계 / 트레이드오프
- <이 설계로 못 막는 것 + scale-trigger(언제 인덱스/샤딩 재검토)>

## 8. 핸드오프
- 구현 계획 → handoffs.plan
- (프로젝트 기존 스키마 문서 시스템 있으면 그 형식으로도 반영)