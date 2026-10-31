# FDE/AX 하네스 일반화: 파일 분류표

지금 `user/`와 `workspace/` 아래에 있는 파일 129개를 FDE/AX용 하네스로 옮길 때 어디에 둘지 나눈 표입니다. 각 파일에서 원래 팀에서만 쓰는 용어(회사명, 사내 repo와 도구 이름, ticket 키, 개인 경로, 사내 디자인 시스템 등)가 나오는 줄 수를 세고, 그 줄이 단순한 예시인지 동작을 정하는 내용인지 읽어서 분류했습니다.

## 분류 기준

| 분류 | 뜻 | 옮길 곳 |
|---|---|---|
| **core** | 고객사가 바뀌어도 그대로 쓴다. 원래 팀 용어가 없거나 예시로만 나온다 | plugin |
| **core·정리** | 동작은 일반적이지만 예시나 하드코딩 값(`<TICKET>-XXXX`, 개인 홈 경로, repo 목록)이 박혀 있다. 그 값을 profile에서 읽게 바꾼다 | plugin |
| **split** | 일반 규칙과 원래 팀 규칙이 한 파일에 섞여 있다. 둘로 나눈다 | core 부분은 plugin, 나머지는 원래 팀 profile |
| **adapter** | 특정 도구나 데이터 소스에 연결하는 코드와 절차다. core는 인터페이스만 정하고, 구현은 도구별로 둔다 | `adapters/<도구>/` |
| **profile** | 원래 팀 과제에서만 의미가 있는 값과 규칙이다 | `profiles/origin-team/` |
| **personal** | 제 개인 환경용이라 FDE 하네스에 넣지 않는다 | `~/.claude/` 그대로 |

## 요약

| 분류 | 파일 수 | 대표 파일 |
|---|---|---|
| core | 55 | debug-investigator, codex-review, triple-review, humanizer, sentry-cli, Codex wrapper와 call template |
| core·정리 | 33 | workflow-01/02/03, review agent 3개, hook 2개, HTML generator 스킬, starter HTML, pr-create |
| split | 9 | `workspace/.claude/CLAUDE.md`, workflow-04 3개, `CODEX.md`, `html-artifact-templates.md`, `TEMPLATE-behavior-fix.md`, `kr-workflow-summary-generator` 2개 |
| adapter | 25 | oncall 스킬과 SQL 19개, `debugger-gui` 2개, `00_CODEX/03_scripts/`의 trace 조회, oncall, 시뮬레이션 triage 스크립트 4개 |
| profile | 3 | `brand-tokens.css`, `TEMPLATE-sprint-sync-brief.md`, `create-jira-from-spec` |
| personal | 4 | `user/.claude/CLAUDE.md`, `cdiff`, `cref`, `aws-support-case` |

core 55개 중 28개는 sentry-cli 설명서입니다. 실제로 손이 가는 파일은 core·정리, split, adapter를 합친 67개이고, 그중 adapter 25개는 내용을 고치지 않고 위치만 옮기면 되니 내용을 고쳐야 하는 파일은 42개입니다.

## user/.claude

| 파일 | 분류 | 근거 | 할 일 |
|---|---|---|---|
| `CLAUDE.md` | personal | 4/86줄. 한국어 작성 원칙, 줄바꿈, em-dash, UTF-8 규칙 | 그대로 둡니다. 고객사 산출물에도 적용하려면 한국어 작성 원칙만 core의 `writing-kr.md`로 복사합니다 |
| `hooks/compact-prep-inject.py`, `compact-prep-remind.py` | core | 0줄 | 그대로 plugin hook으로 옮깁니다 |
| `skills/compact-prep/SKILL.md` | core·정리 | 1/125줄. ticket 키 예시 하나 | 예시를 `<TICKET>`으로 바꿉니다 |
| `skills/prompt-check/` | core | 0줄 | AX 과제에서 고객사 prompt를 다듬을 때 그대로 씁니다 |
| `skills/humanizer/` (3개) | core | 0줄 | 고객 보고서 문장 점검용으로 그대로 씁니다 |
| `skills/humanize-kr/` (2개) | core·정리 | 4줄씩. 예문에 원래 팀 용어 | 예문을 업종 중립적인 문장으로 바꿉니다 |
| `skills/sentry-cli/` (28개) | core | 0줄 | 공개 도구 설명서라 그대로 씁니다. 고객사가 Sentry를 쓰지 않으면 켜지 않습니다 |
| `skills/cdiff/`, `skills/cref/` | personal | 0줄이지만 특정 터미널 앱과 개인 도구를 전제로 함 | 넣지 않습니다 |
| `skills/aws-support-case/` | personal | 2줄. 회사명과 이름이 템플릿에 박힘 | 넣지 않습니다. 고객사 AWS를 다루는 과제가 생기면 그때 profile 값으로 바꿔 adapter로 만듭니다 |

## workspace/.claude

| 파일 | 분류 | 근거 | 할 일 |
|---|---|---|---|
| `CLAUDE.md` | split | 36/318줄. 위쪽 Workflow Overview, Planning Depth, Agents, Agent Teams, Templates는 일반적이고, 진입점 3개, team repo 규칙, Directory Structure, Testing, Jira Query는 원래 팀 전용 | core `CLAUDE.md`(track, depth, agent, workflow 파일 표)와 profile `CLAUDE.md`(repo, 진입점, 테스트 도구, ticket)로 나누고, profile이 core를 `@import`합니다 |
| `settings.json` | core·정리 | hook 2개는 일반적이고, 사내 문서 도구 MCP 권한 하나만 원래 팀 쪽 | hook은 plugin으로, permission은 profile `settings.json`으로 옮깁니다 |
| `agents/code-reviewer.md` | core·정리 | 1/106줄. ticket 키 예시 | 예시만 바꿉니다 |
| `agents/pr-reviewer.md` | core·정리 | 1/152줄. ticket 키 예시 | 예시만 바꿉니다 |
| `agents/debug-investigator.md` | core | 0줄 | 그대로 씁니다 |
| `agents/codex-result-auditor.md` | core·정리 | 2/147줄. Gate 6(local-path leakage)이 개인 홈 경로와 사내 repo 이름을 직접 적음 | 금지 경로와 허용 경로를 profile에서 읽게 바꿉니다. 이 Gate를 넓혀서 "다른 고객사 이름이나 데이터가 산출물에 들어갔는지"도 보게 하면, 고객사 간 격리 검사로 그대로 쓸 수 있습니다 |
| `hooks/codex-guard.sh` | core·정리 | 1/93줄. 제외할 repo 목록이 주석과 코드에 박힘 | 제외 목록을 `engagement.json`에서 읽습니다 |
| `hooks/md-join-test.sh` | core·정리 | 5/109줄. 같은 구조로 team repo 경로를 제외함 | 위와 같습니다 |
| `skills/codex-review/` (5개) | core | 0줄 | 그대로 씁니다 |
| `skills/codex-teamreview/` (3개) | core·정리 1, core 2 | SKILL.md에 ticket 키 예시 1줄, `references/` 2개는 0줄 | SKILL.md 예시만 바꿉니다 |
| `skills/teamreview/` | core·정리 | 1/155줄 | 예시만 바꿉니다 |
| `skills/triple-review/` | core | 0줄 | 그대로 씁니다 |
| `skills/review-html-generator/` (4개) | core·정리 | SKILL.md 3줄, `template.html` 4줄. 출력 경로에 개인 경로, 템플릿에 사내 디자인 토큰 | 경로는 profile의 `docs_root`로, 디자인 토큰은 아래 디자인 토큰 slot으로 바꿉니다. `scripts/` 2개는 0줄이라 그대로 씁니다 |
| `skills/primer-html-generator/` | core·정리 | 6/93줄 | 같습니다. Handover 단계에서 고객 팀용 primer를 만드는 데 쓰니 우선순위가 높습니다 |
| `skills/runbook-html-generator/` | core·정리 | 5/44줄 | 같습니다. Handover 산출물입니다 |
| `skills/deepdive-html-generator/` | core·정리 | 4/45줄. 회사명과 제품명 예시, 사내 디자인 토큰 | 같습니다 |
| `skills/demo-site/` | core·정리 | 3/70줄. 개인 경로, 사내 디자인 토큰 | 같습니다. PoC 시연에 씁니다 |
| `skills/share-as-artifact/` (2개) | core·정리 | SKILL.md 4줄, 스크립트 3줄. ticket 키 예시, 사내 디자인 토큰 | 같습니다. 고객사에 공유할 때 private 기본값이 맞는지 profile의 공유 규칙으로 한 번 더 확인하게 합니다 |
| `skills/pr-create/` | core·정리 | 5/85줄. `[<TICKET>-XXXX] title` 형식이 박힘 | PR 제목 형식을 `engagement.json`의 `pr.title_format`에서 읽게 바꿉니다 |
| `skills/kr-workflow-summary-generator/` (2개) | split | SKILL.md 3줄. 요약 틀은 일반적이고, ticket 키와 업무 지침 문서 분류만 원래 팀 쪽 | 틀은 core의 주간 stakeholder 보고 스킬로 넓히고, ticket 분류 규칙은 profile로 옮깁니다 |
| `skills/create-jira-from-spec/` | profile | 5/85줄. ticket project와 필드가 원래 팀 설정 | ticket adapter의 구현 하나로 옮기고, core에는 "spec에서 ticket 만들기" 인터페이스만 둡니다 |
| `skills/debugger-gui/` (2개) | adapter | 19/74줄, 10/332줄. 팀 디버거 도구 전용 | `adapters/origin-sim-test/`로 옮깁니다 |
| `skills/oncall/SKILL.md`, `references/alert-routing.md` | adapter | 49/161줄, 20/94줄. 사내 알럿 도구와 대시보드 | `adapters/origin-oncall/`로 옮깁니다. "알럿 받기 → 쿼리 → 원인 분류 → 밖으로 나가는 행동 없이 보고"라는 절차는 core의 운영 단계 workflow로 따로 적어 둡니다 |
| `skills/oncall/references/queries/*.sql` (17개) | adapter | 대부분 0~2줄이지만 테이블 스키마가 원래 팀 제품 전용 | 위와 같이 옮깁니다 |

## workspace/00_CLAUDE

| 파일 | 분류 | 근거 | 할 일 |
|---|---|---|---|
| `workflow-01-planning.md` | core·정리 | 7/525줄. Depth, Spec 인터뷰, Explore, Dev Plan, Codex 토론 모두 일반적이고, Step 0의 대상 repo 고르기만 사내 repo 이름을 적음 | 대상 repo 목록을 profile에서 읽게 바꿉니다. FDE용으로는 Phase 1 앞에 Discovery를 붙입니다(아래 "새로 만들 것") |
| `workflow-02-implementation.md` | core·정리 | 8/420줄. worktree 설정과 팀 디버거 repo 언급 | base branch와 worktree 규칙을 profile로 옮깁니다 |
| `workflow-03-documentation.md` | core·정리 | 6/127줄. knowledge 문서 분류 예시가 업무 지침 문서, 시뮬레이션 테스트 | 예시를 바꾸고, lesson을 core로 올릴지 profile에 둘지 정하는 규칙을 추가합니다 |
| `workflow-04-delivery.md` | split | 45/228줄. track 선택, D0~D5 순서, D1의 결과 5종, 3단 보고, evidence tier는 일반적이고, 진입점, SOP gate, 특정 고객사 예외, 팀 스킬 위임, D2 layer 이름은 원래 팀 전용 | core의 "behavior track" workflow로 다시 씁니다. layer 이름은 retrieval, routing, main-LLM 분기, prompt는 실행됐지만 결과가 틀림, model·template, 동작 변경 같은 일반 이름으로 두고, adapter가 원래 팀의 파일 이름(routing 지침 파일, KB, 업무 지침 문서)과 연결합니다 |
| `workflow-04-delivery-verify.md` | split | 19/87줄. "가장 싼 증거부터 쓰고, 아래 단계가 통과해야 위로 간다"는 rung 구조는 일반적이고, rung 내용은 사내 대화 시뮬레이션 도구와 로컬 agent 실행 환경 | rung 1~4를 `eval-runner` adapter의 `replay`, `local`, `staged`, `regression` 동작으로 바꿉니다 |
| `workflow-04-delivery-ship.md` | split | 16/97줄. test asset과 PR 기록 틀은 일반적이고, 에이전트 정의 repo의 PR 형식은 원래 팀 전용 | 틀만 core에 두고 PR 형식은 profile로 옮깁니다 |
| `09_templates/TEMPLATE-spec.md` | core·정리 | 2/97줄 | 예시만 바꿉니다 |
| `09_templates/TEMPLATE-dev-plan.md` | core | 0줄 | 그대로 씁니다 |
| `09_templates/TEMPLATE-debugging.md` | core | 0줄 | 그대로 씁니다 |
| `09_templates/TEMPLATE-knowledge.md` | core·정리 | 1/120줄 | 예시만 바꿉니다 |
| `09_templates/TEMPLATE-behavior-knowledge.md` | core·정리 | 2/71줄 | 예시만 바꿉니다 |
| `09_templates/TEMPLATE-behavior-fix.md` | split | 7/106줄. 진단 노트 틀은 일반적이고, 시뮬레이션 테스트 링크와 사내 테스트 도구 필드가 박힘 | "eval run 링크", "실행 환경" 같은 일반 필드로 바꾸고, 원래 팀 필드는 profile 템플릿에서 덧붙입니다 |
| `09_templates/TEMPLATE-sprint-sync-brief.md` | profile | 4/87줄. 고객 ticket과 사내 테스트 도구 지표 기준 | profile로 옮깁니다. FDE용 주간 보고 템플릿은 새로 만듭니다 |
| `09_templates/html-artifact-templates.md` | split | 15/109줄. starter 고르는 법은 일반적이고, 사내 디자인 시스템 사용 규칙은 원래 팀 쪽 | 고르는 법은 core에, 브랜드 규칙은 profile에 둡니다 |
| `09_templates/html/brand-tokens.css` | profile | 22/220줄. 원래 팀의 사내 디자인 시스템 | profile로 옮기고, core에는 중립 토큰 파일 `tokens.css`를 둡니다. 고객사 브랜드로 보고서를 내야 하면 그 고객사 profile이 자기 토큰 파일을 넣습니다 |
| `09_templates/html/preview.html` | core·정리 | 9/146줄. 사내 디자인 토큰 미리보기 | 토큰 파일을 바꿔 끼울 수 있게 합니다 |
| `09_templates/html/starter-*.html` (5개) | core·정리 | 3~5줄씩. 사내 디자인 토큰 링크와 예시 문구 | 토큰 파일 경로를 바꾸고 예시 문구를 중립적으로 바꿉니다. slidedeck과 report는 고객 보고에 바로 씁니다 |
| `09_templates/html/starter-docsite/` (5개) | core·정리 | `index.html`만 4줄, 나머지 0줄 | `index.html`의 토큰 경로만 바꿉니다 |

## workspace/00_CODEX

| 파일 | 분류 | 근거 | 할 일 |
|---|---|---|---|
| `CODEX.md` | split | 15/62줄. 앞부분이 workspace 구조와 원래 팀 제품 설명 | Codex에게 주는 공통 규칙은 core에, 과제 설명은 profile에서 만들어 붙입니다 |
| `01_prompts/codex-call-templates.md` | core | 0/354줄 | 그대로 씁니다 |
| `03_scripts/codex-run.sh` | core | 0/266줄 | 그대로 씁니다. 다만 `CLAUDE.md`에 있는 Codex 계정 규칙(회사 계정 종류, 할당량 공유)은 profile로 옮깁니다. 고객사에 따라 외부 모델 사용 자체가 막힐 수 있으니 profile에 `codex.enabled`를 둡니다 |
| `03_scripts/codex-model-check.py`, `codex-quota-check.py` | core | 0줄 | 그대로 씁니다 |
| `03_scripts/trace-fetch` 스크립트 | adapter | 17/151줄 | `trace-fetch` adapter의 BigQuery 구현으로 옮깁니다 |
| `03_scripts/oncall-query.sh` | adapter | 32/370줄. 사내 로그 DB 접속 정보 | `adapters/origin-oncall/`로 옮깁니다 |
| `03_scripts/oncall-triage.py` | adapter | 21/954줄 | 같습니다 |
| `03_scripts/sim-triage.py` | adapter | 28/521줄 | `adapters/origin-sim-test/`로 옮깁니다 |

## 새로 만들 것

분류표에는 없지만, 위 분류대로 나누면 바로 필요해지는 것입니다.

| 항목 | 이유 |
|---|---|
| `engagement.json` 스키마 | `team.json`을 넓힌 것입니다. 고객사 이름, repo와 base branch, ticket prefix와 PR 형식, `docs_root`, 권한 경계(proposer인지 deployer인지), 외부 모델 허용 여부, hook 제외 경로를 담습니다. core·정리로 분류한 22개 파일이 모두 여기서 값을 읽습니다 |
| adapter 인터페이스 3개 | `eval-runner`(replay, local, staged, regression), `trace-fetch`(대화나 실행 기록 가져오기), `ticket`(조회, 생성, PR 연결). workflow-04 3개 파일이 이 이름만 부르게 됩니다 |
| Discovery, Pilot gate, Handover workflow | 지금 하네스에는 "이미 운영 중인 시스템을 고치는" 단계만 있습니다. FDE/AX 과제의 앞(업무 흐름 정리, 합격 기준과 eval set 합의)과 뒤(고객 팀에 넘기기)가 비어 있습니다 |
| 고객사 간 격리 hook | 한 고객사 profile에서 일하는 동안 다른 고객사 이름, 경로, 데이터가 산출물에 들어가지 않게 막습니다. `codex-result-auditor`의 Gate 6을 바탕으로 만듭니다 |
| `/engagement-init` 스킬 | 새 고객사를 시작할 때 인터뷰해서 `engagement.json`과 profile `CLAUDE.md`를 만듭니다 |
| `profiles/origin-team/` | 위에서 profile과 split으로 나온 원래 팀 부분을 모은 첫 profile입니다. 일반화한 core와 이 profile을 합쳤을 때 지금과 같은 동작이 나오는지로 일반화가 맞았는지 확인합니다 |

## 작업 순서

```
1. engagement.json 스키마 정의
   └─ core·정리 33개가 읽을 값을 먼저 정해야 손댈 수 있음
2. core·정리 22개 정리 (값을 profile에서 읽게)
3. split 7개 분리, 특히 CLAUDE.md와 workflow-04
   └─ 여기서 adapter 인터페이스 이름이 정해짐
4. adapter 25개를 adapters/origin-*/로 이동
5. profiles/origin-team/ 구성 후 지금 동작과 비교
6. Discovery, Handover workflow와 격리 hook, /engagement-init 추가
```

1~5번은 원래 팀 과제에서 지금처럼 동작하는지를 기준으로 확인할 수 있습니다. 6번은 비교할 기준이 없으니 실제 FDE 과제 하나에 적용해 보면서 다듬어야 합니다.
