# persona-suite

개발 라이프사이클 단계별로 **시니어 페르소나 티키타카**를 돌려, 실제 피처를 정의/설계하면서 동시에 프로세스를 체화하는 Claude Code 플러그인.

여러 프로젝트·여러 컴퓨터에서 쓰기 위해 플러그인으로 배포한다. 프로젝트별 specifics(용량 제약·경로·관례)는 플러그인이 아니라 각 repo의 `.claude/persona-profile.md` 에 둔다.

## 왜 만들었나

혼자 개발하는 환경에는 요구사항과 설계의 빈틈을 짚어줄 시니어·리뷰어가 없다 — 빈틈은 구현이 끝난 뒤에야 드러난다.
그 압박을 사람 대신 라이프사이클 단계별 시니어 페르소나(PM·리뷰어·DBA·SRE·보안…)의 티키타카로 시스템화했다.
실제 피처의 산출물을 페르소나가 압박해 확정하고, 그 과정을 반복하며 프로세스 자체를 체화한다. (구현·테스트 단계의 같은 문제는 자매 repo [claude-agents](https://github.com/xxx-sj/claude-agents)가 맡는다.)

## 구성 (skills)

| skill | 단계 | 상태 |
|---|---|---|
| `defining-requirements` | 요구사항 정의 (①요구사항→②기능정의→③기술설계) | ✅ |
| `designing-data-model` | 상세 스키마/인덱스 설계 ([REV][DBA][DATA][SRE][SEC]) | ✅ |
| `designing-code` | 모듈/코드 설계 ([ARCH][REV][SEC]) | ✅ v1 — light 검증 / heavy 미검증 |
| `validating-test-lists` | 테스트 리스트업 (빠진/불필요/엣지 검증) | ✅ |
| `auditing-production-risks` | 구현 후 프로덕션 리스크 감사 (독립 서브에이전트 판정표) | ✅ v1 — 미검증 (dogfood 대기) |

설계 근거는 [`DESIGN.md`](./DESIGN.md) (검증 이력은 그 안 dogfood 로그). 전체 개발 라이프사이클 + 각 스킬 위치는 [`LIFECYCLE.md`](./LIFECYCLE.md). 페르소나 스킬 공통 원칙(단일 출처)은 [`PRINCIPLES.md`](./PRINCIPLES.md). 완성 산출물 예시는 [`examples/`](./examples/) (defining-requirements·designing-data-model — 같은 가공 피처의 연속 산출물).

## 호출 — 자연어 자동 / 슬래시 커맨드(명시·결정적)

각 스킬은 자연어 키워드로 자동 표면화되지만, **결정적으로 특정 단계만** 부르려면 슬래시 커맨드 (`commands/`). 각 커맨드는 *자기 단계만* 실행하고 **다음 단계를 자동 호출하지 않는다** — 다음은 `LIFECYCLE.md` 안내 따라 직접 호출:

| 커맨드 | 호출 스킬 | 단계 |
|---|---|---|
| `/persona-suite:define-requirements` | `defining-requirements` | 1~3 요구사항·기능정의·기술설계(고수준) |
| `/persona-suite:design-data-model` | `designing-data-model` | 4 데이터모델 |
| `/persona-suite:design-code` | `designing-code` | 5 코드설계 |
| `/persona-suite:validate-test-lists` | `validating-test-lists` | 6b 테스트리스트업 |
| `/persona-suite:audit-production-risks` | `auditing-production-risks` | 8.5 프로덕션 리스크 감사 |

## 설치 (각 컴퓨터에서)

GitHub 등에 이 repo를 push 한 뒤:

```
/plugin marketplace add xxx-sj/persona-suite
/plugin install persona-suite@persona-suite
```

로컬 테스트(push 전):

```
/plugin marketplace add ~/Documents/workspace/projects/persona-suite
/plugin install persona-suite@persona-suite
```

## 업데이트

```
# 개발 머신
git commit -am "..." && git push

# 다른 머신
/plugin marketplace update persona-suite
```

`plugin.json` 에 `version` 을 두지 않아 매 커밋이 새 버전으로 잡힌다 (개발 단계 편의). 릴리즈를 고정하려면 `version` 추가.

## 사용 (defining-requirements)

트리거: "이 피처 요구사항 정의", "스펙 잡자", "요구사항 정리" 등으로 자동 표면화. 또는 특정 페르소나만 개별 호명("지금 PM 페르소나로만").

흐름:
1. 실제 피처 한 줄 제시 → 앵커 고정 + 신규/수정 판별 + 무게 tier.
2. (수정이면) 현행 코드 조사로 AS-IS 맵.
3. `[PM]`→`[REV]`→`[TECH]` 단방향 압박, 빈틈은 한 칸 위로 역류.
4. 재화/권한이면 `[SEC]`, 운영 surface면 `[OPS]`, 마지막 `[QA]`가 AC.
5. `[REV]` 종료 완성도 패스 → 요구사항 문서 산출 → 테스트 리스트/구현 계획으로 핸드오프.

### 프로젝트 프로필

각 repo의 `.claude/persona-profile.md` 에 용량 제약·출력 위치·핸드오프 타깃·횡단 정책을 둔다. **없으면 첫 실행 때 스킬이 자동 생성**한다 (CLAUDE.md 등에서 추론 + 빈칸만 질문 → 파일로 저장). 프로필이 없어도 스킬은 물으면서 동작한다.