# persona-suite

개발 라이프사이클 단계별로 **시니어 페르소나 티키타카**를 돌려, 실제 피처를 정의/설계하면서 동시에 프로세스를 체화하는 Claude Code 플러그인.

여러 프로젝트·여러 컴퓨터에서 쓰기 위해 플러그인으로 배포한다. 프로젝트별 specifics(용량 제약·경로·관례)는 플러그인이 아니라 각 repo의 `.claude/persona-profile.md` 에 둔다.

## 구성 (skills)

| skill | 단계 | 상태 |
|---|---|---|
| `defining-requirements` | 요구사항 정의 (①요구사항→②기능정의→③기술설계) | ✅ |
| `designing-data-model` | 상세 스키마/인덱스 설계 ([REV][DBA][DATA][SRE][SEC]) | ✅ |
| `designing-code` | 모듈/코드 설계 | 예정 |

설계 근거는 [`DESIGN.md`](./DESIGN.md).

## 설치 (각 컴퓨터에서)

GitHub 등에 이 repo를 push 한 뒤:

```
/plugin marketplace add <github-owner>/persona-suite
/plugin install persona-suite@persona-suite
```

로컬 테스트(push 전):

```
/plugin marketplace add ~/Documents/workspace/projects/persona-suite
/plugin install persona-suite@persona-suite
```

> `<github-owner>` 는 본인 GitHub 계정으로 교체.

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