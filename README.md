# generalized-flows


## 구성

```
korean/
  CLAUDE.md              응답 방식, 한국어 작성 원칙, 줄바꿈·em-dash·UTF-8 규칙
  humanize-kr/           한국어 AI 문체를 찾아 고치는 스킬 (SKILL.md, patterns.md)
workflows/
  workflow-01-planning.md         T track Phase 0~2: depth, spec 인터뷰, explore, dev plan
  workflow-02-implementation.md   T track Phase 3~3.6: 구현, 리뷰, 디버깅, 검증 체크리스트
  workflow-03-documentation.md    공통 Phase 4~5: knowledge 문서, lessons
  workflow-04-delivery.md         D track D0~D3.5: 에이전트 동작으로 판정하는 작업
  workflow-04-delivery-verify.md  D track D4: 검증 단계와 결과 읽는 법
  workflow-04-delivery-ship.md    D track D5: 테스트 자산 정리, PR, 기록
templates/
  TEMPLATE-spec.md, TEMPLATE-dev-plan.md, TEMPLATE-debugging.md, TEMPLATE-knowledge.md
  TEMPLATE-behavior-fix.md, TEMPLATE-behavior-knowledge.md, TEMPLATE-sprint-sync-brief.md
inventory.md             원래 하네스 전체를 일반화할 때 어디에 둘지 나눈 분류표 (참고용)
```

## 두 가지 track

작업의 정답을 무엇으로 판정하느냐에 따라 track이 정해집니다.

```
코드 테스트로 판정    → T track: workflow-01 → 02 → 03
에이전트 대화로 판정  → D track: workflow-04 (D0 → D5) → 03
둘 다                → 나눔: 코드는 T, 고객사별 검증은 D
```

D track의 도구 이름(eval platform, sim profile, scenario case, staged run, trace store 등)은 일반 이름입니다. `workflow-04-delivery.md` 앞부분의 Vocabulary 표에 뜻과 예시가 있으니, 과제를 시작할 때 그 과제의 실제 도구와 짝지어 두면 됩니다.

## 설치

workflow 파일은 템플릿과 문서 위치를 `00_CLAUDE/` 기준 상대 경로로 가리킵니다. 그대로 쓰려면 이렇게 둡니다.

```
~/.claude/CLAUDE.md                 ← korean/CLAUDE.md 내용을 추가
~/.claude/skills/humanize-kr/       ← korean/humanize-kr/
<workspace>/00_CLAUDE/workflow-*.md ← workflows/
<workspace>/00_CLAUDE/09_templates/ ← templates/
<workspace>/.claude/team.json       ← ticket prefix, base branch, PR 형식, 다른 팀 repo 목록 (직접 작성)
```

그리고 workspace의 `.claude/CLAUDE.md`에 "작업을 받으면 track을 정하고, 해당 workflow 파일을 먼저 읽는다"는 규칙과 workflow 파일 표를 적어 둡니다.

## 포함하지 않은 것

- 스킬과 agent(`code-reviewer`, `pr-reviewer`, `teamreview`, `codex-review`, `review-html-generator` 등): workflow 파일이 이름으로 부르지만 이 repo에는 없습니다. 없는 단계는 건너뛰거나 직접 하면 됩니다.
