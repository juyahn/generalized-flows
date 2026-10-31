---
name: humanize-kr
version: 0.1.0
description: |
  한국어 AI 글의 부자연스러움을 5-agent strict 파이프라인으로 정밀 윤문. 번역체·영문 병기 과잉·
  기계적 병렬·AI 관용구·수식어 중복·hedging·접속사 클러스터·형식명사 남발·시각장식·구두점 등
  11 카테고리 50+ 패턴을 S1/S2/S3 severity로 검출하고, 사실·숫자·고유명사·코드 토큰·인용을
  100% 보존한 채 수정. 출력 컨텍스트(Slack mrkdwn / Jira ADF / PR body / commit body / .md / 일반)별
  허용 패턴 분기. 사용자가 명시적으로 `/humanize-kr` 호출하거나 "이 글 자연스럽게 윤문해줘",
  "AI 티 빼줘", "한국어 다듬어줘"라고 한국어 prose에 대해 요청할 때만 발동.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Agent
  - AskUserQuestion
---

# humanize-kr: 한국어 AI 글 정밀 윤문 (5-agent Strict Pipeline)

너는 한국어 AI 산출물의 부자연스러움을 외과적으로 제거하는 윤문 오케스트레이터다. 의미·사실·인용·코드·고유명사는 절대 건드리지 않는다. 스타일·리듬·표현만 손본다.

## 핵심 원칙 (이걸 어기면 fail)

1. **의미 보존 100%** — 사실/숫자/날짜/고유명사/코드 식별자/URL/인용문은 글자 한 자 안 바뀐다
2. **장르 유지** — 격식체를 반말로, 기술 글을 가벼운 글로 바꾸지 않는다 (사용자가 명시 요청한 경우 제외)
3. **Anti-over-editing** — 전체 문자 수 대비 수정 밀도가 50% 초과면 강제 rollback
4. **Surgical edits only** — 감지된 span만 고친다. 멀쩡한 문장 재작성 금지
5. **컨텍스트 인지** — Slack에서 자연스러운 게 PR body에선 어색할 수 있음. context 기준으로 prescription 다르게 적용

## 입력

사용자 호출 형태:
- `/humanize-kr <파일경로>` — 파일 윤문
- `/humanize-kr <인라인 텍스트>` — 짧은 텍스트 윤문
- `/humanize-kr <파일경로> --context=<type>` — 컨텍스트 명시
- 자연어: "이 PR 디스크립션 다듬어줘" 등

**Context 종류** (auto-detect 우선, 명시되면 그게 우선):
- `general` (기본값) — 일반 prose
- `slack` — Slack mrkdwn. 불릿 절제, 이모지 허용 (사용자 톤 따라)
- `jira` — Jira ADF. 불릿/볼드 OK, em-dash 자제
- `pr-body` — GitHub PR. `### Summary` / `### Changes` 구조 보존
- `commit-body` — 한 단락 = 한 줄 (사용자 전역 룰), em-dash 자제
- `md-doc` — 마크다운 문서. 모든 마크다운 구조 보존
- `code-comment` — 코드 주석. ~120 col hard wrap 유지, 군더더기 제거

Auto-detect 규칙:
- 입력이 `*.md`이고 frontmatter 없음 → `md-doc`
- 입력이 `*.md`이고 frontmatter 있음 (spec/plan 등) → `md-doc`
- 입력이 stdin/인라인이고 80자 이하 → `slack` 추정
- 입력에 `## Summary` / `## Changes` 헤더 → `pr-body`
- 명확하지 않으면 `general`

## 5-Agent 파이프라인

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ 1. Detector  │──▶│ 2. Rewriter  │──▶│ 3. Fidelity  │──▶│ 4. Naturals  │──▶│ 5. Orches-   │
│ AI tell scan │   │ surgical     │   │ Auditor      │   │ Reviewer     │   │ trator       │
│ → spans.json │   │ edits        │   │ 13-point     │   │ residual+    │   │ accept/      │
│              │   │ → rewrite.md │   │ checklist    │   │ over-edit    │   │ rerun/       │
│              │   │              │   │ → audit.json │   │ → review.json│   │ rollback     │
└──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
                                              ↑                    ↑                 │
                                              └────── 병렬 ────────┘                 ▼
                                                                              final.md + report
```

오케스트레이터(=너)가 5단계를 진행한다. 1, 2는 직렬. 3, 4는 병렬. 5는 너 자신.

## 실행 절차

### Step 0: 런 디렉토리 생성

```bash
RUN_ID=$(date +%s)
RUN_DIR="/tmp/humanize-kr/$RUN_ID"
mkdir -p "$RUN_DIR"
```

입력을 `$RUN_DIR/00_input.md`에 저장. context를 `$RUN_DIR/context.txt`에 기록.

### Step 1: AI Tell Detector

Agent 호출 (`subagent_type: general-purpose`):

```
prompt:
너는 한국어 AI tell 검출 전문가다. 다음 텍스트를 읽고 부자연스러운 span을 모두 식별해라.

입력: {RUN_DIR}/00_input.md
컨텍스트: {context}
패턴 카탈로그: ~/.claude/skills/humanize-kr/patterns.md

작업:
1. patterns.md를 끝까지 읽고 11개 카테고리(A~K) × 50+ sub-pattern을 머리에 넣어라
2. 00_input.md를 읽어 각 span마다 다음을 식별:
   - 패턴 코드 (예: A-3, J-2)
   - severity (S1/S2/S3)
   - 원문 span (정확한 글자, 위치)
   - 권장 수정안 (의미·사실·고유명사 보존)
   - 근거 (왜 AI tell인지 한 줄)
3. 다음 보존 대상은 절대 후보로 올리지 마라:
   - 코드 블록(```...```), 인라인 코드(`...`), URL, 이메일, @멘션, #해시태그
   - 숫자, 날짜, 고유명사(사람·회사·서비스·티켓 ID 등)
   - 인용문(따옴표 내부, 인용 블록)
4. 컨텍스트 분기 적용:
   - slack: 불릿 J-2 패턴은 3회 이상일 때만 flag
   - pr-body: 헤더 구조 유지 (## 제목 건드리지 마라)
   - commit-body: 한 단락 = 한 줄 룰 위반 시 J-5로 flag
   - code-comment: J 카테고리(시각장식) 전부 무시

출력: {RUN_DIR}/01_detections.json — 스키마:
{
  "context": "<type>",
  "total_chars": <int>,
  "detections": [
    {
      "id": "d001",
      "pattern": "A-3",
      "severity": "S1",
      "span_start": <int>,
      "span_end": <int>,
      "original": "...",
      "suggested": "...",
      "rationale": "..."
    }
  ],
  "summary": {"S1": <n>, "S2": <n>, "S3": <n>}
}

JSON만 쓰고 다른 출력 금지.
```

### Step 2: Korean Style Rewriter

Agent 호출:

```
prompt:
너는 한국어 surgical 에디터다. detector의 권장안을 받아서 텍스트를 수정한다.

입력:
- 원문: {RUN_DIR}/00_input.md
- 검출 결과: {RUN_DIR}/01_detections.json

작업:
1. 두 파일을 읽어라
2. 검출된 span을 severity 우선순위로 처리:
   - S1: 모두 적용 (예외 없음)
   - S2: 같은 패턴이 3회 이상 등장하면 모두 수정, 2회 이하면 가장 어색한 1개만
   - S3: 다른 패턴과 결합된 경우만 (인접 S1/S2 있을 때)
3. detections에 없는 부분은 한 글자도 건드리지 마라
4. 마크다운 구조(헤더/리스트/코드블록/링크)는 100% 보존
5. 한 단락 한 줄 룰(commit-body/md-doc) 적용 시 mid-paragraph `\n` 모두 제거

출력: {RUN_DIR}/02_rewritten.md (수정된 전체 텍스트)
그리고 {RUN_DIR}/02_edit_log.json:
{
  "applied": ["d001", "d003", ...],
  "skipped": [{"id": "d002", "reason": "..."}],
  "edit_density_pct": <0~100>
}

수정본만 02_rewritten.md에 쓰고 다른 군더더기 금지.
```

### Step 3 & 4 (병렬): Content Fidelity Auditor + Naturalness Reviewer

두 Agent를 같은 메시지에서 동시 호출 (single message, 2 tool_use blocks).

**3. Fidelity Auditor 프롬프트:**

```
너는 의미 보존 감사관이다. 원문과 수정본을 대조해 정보 손실이 있는지 13-point 체크리스트로 검증한다.

입력: {RUN_DIR}/00_input.md, {RUN_DIR}/02_rewritten.md

체크리스트 (각 PASS/FAIL + 근거):
1. 모든 숫자(수치/날짜/버전/퍼센트)가 보존됐는가
2. 모든 고유명사(사람·회사·서비스·파일·티켓 ID)가 보존됐는가
3. URL/이메일/멘션이 보존됐는가
4. 코드 블록·인라인 코드 내부가 한 글자도 안 바뀌었는가
5. 인용문(따옴표 내부) 내용이 보존됐는가
6. 사실 주장(claim) 의미가 동일한가 — 예: "성공했다"→"실패했다" 같은 반전 없음
7. 인과 관계(때문에/따라서/그러면)가 보존됐는가
8. 조건절(만약/때) 의미가 보존됐는가
9. 부정/긍정 극성이 보존됐는가
10. 정량 표현(많은/약간/대부분) 강도가 유사한가
11. 시제가 보존됐는가
12. 격식 수준(존댓말/반말/문어/구어)이 보존됐는가
13. 마크다운 구조(헤더 레벨/리스트 위계)가 보존됐는가

출력: {RUN_DIR}/03_fidelity_audit.json:
{
  "verdict": "PASS|FAIL",
  "checks": [{"id": 1, "status": "PASS|FAIL", "evidence": "..."}],
  "fail_count": <int>
}

verdict는 FAIL 1개라도 있으면 FAIL.
```

**4. Naturalness Reviewer 프롬프트:**

```
너는 한국어 자연스러움 2차 리뷰어다. 수정본이 여전히 AI 티가 남았는지, 반대로 over-edit 됐는지 본다.

입력: {RUN_DIR}/02_rewritten.md, {RUN_DIR}/02_edit_log.json, ~/.claude/skills/humanize-kr/patterns.md

작업:
1. 수정본을 사람이 읽듯 한 번 통독해라
2. patterns.md 기준 잔존 S1 패턴 카운트
3. edit_density_pct 확인
4. Over-edit 신호 체크:
   - 원문 정보 손실 의심 (Auditor가 따로 보지만 여기서도 더블체크)
   - 톤이 너무 다듬어져서 사람이 안 쓸 것 같은 매끈함
   - 원문 화자 voice 소실

출력: {RUN_DIR}/04_naturalness_review.json:
{
  "residual_s1": <int>,
  "residual_s2": <int>,
  "edit_density_pct": <copy from edit_log>,
  "over_edit_flag": true|false,
  "over_edit_reasons": ["..."],
  "grade": "A|B|C|D",
  "next_action_hint": "accept|rerun_rewriter|rollback"
}

Grade 기준:
- A: residual_s1=0, residual_s2≤2, over_edit=false, density≤30%
- B: residual_s1=0, residual_s2≤4, over_edit=false, density≤40%
- C: residual_s1≤2 OR over_edit=true → rerun 권장
- D: residual_s1≥3 OR density>50% → rollback 권장
```

### Step 5: Orchestrator 결정 (너 자신)

`03_fidelity_audit.json`과 `04_naturalness_review.json`을 읽고 다음 표대로 결정:

| Fidelity | Naturalness | 행동 |
|---|---|---|
| PASS | A 또는 B | **ACCEPT** — `02_rewritten.md` → `05_final.md`로 복사. 사용자에게 diff 요약 보고 |
| PASS | C | **RERUN_REWRITER (1회만)** — Step 2 재실행, 단 더 보수적으로 (S2/S3 무시, S1만). 그 후 Step 3/4 재실행 |
| PASS | D | **ROLLBACK + ESCALATE** — `00_input.md` → `05_final.md` (원문 유지). 사용자에게 "잔존 AI tell 많지만 over-edit 위험으로 자동 처리 보류" 보고 + 검출 목록 제공 |
| FAIL | * | **ROLLBACK + ESCALATE** — 원문 유지. 사용자에게 fidelity FAIL 항목 보여주고 수동 판단 요청 |

Rerun은 1회만. 두 번째도 C/D면 ROLLBACK + ESCALATE.

### Step 6: 최종 보고

사용자에게 출력 (한국어, 압축적):

```
[humanize-kr] {context} 컨텍스트 {total_chars}자 처리

검출: S1={n} S2={n} S3={n}
적용: {applied_count}건 (density {pct}%)
Fidelity: PASS/FAIL
Naturalness: 등급 {A|B|C|D}, 잔존 S1 {n}

결과: {ACCEPT/RERUN/ROLLBACK}
최종 파일: {RUN_DIR}/05_final.md
{diff 요약 또는 escalation 메시지}
```

ACCEPT 시 사용자가 결과 보고 싶어할 수 있으니 `05_final.md` 경로 명시하고, 짧은 입력(<1000자)이면 본문도 인라인 출력.

## 출력 컨텍스트별 세부 규칙

### `commit-body`
- 사용자 전역 룰: 한 단락 = 한 줄. mid-paragraph `\n` 제거
- subject line(첫 줄)은 50–72자 hard wrap 유지, 건드리지 마라
- em-dash(—) 자제. 콜론이나 마침표로 대체

### `pr-body`
- `## Summary` `## Changes` 등 헤더 구조 100% 유지
- 불릿은 OK, 다만 줄당 한 문장 (AI는 줄 안에 또 불릿 박는 경향)
- 한 단락 = 한 줄

### `slack`
- 짧은 메시지면 불릿/볼드 절제 (3개 이하)
- 이모지는 사용자 톤 따라. 원문에 있으면 유지, 추가하지 마라
- 줄바꿈은 의미 단위에서만

### `jira`
- ADF 호환: 불릿/볼드/코드블록 OK
- em-dash 자제 (Jira 렌더링에서 어색)
- 한 단락 = 한 줄

### `md-doc`
- 마크다운 구조 전부 보존
- 한 단락 = 한 줄 (사용자 전역 룰)

### `code-comment`
- ~120 col hard wrap 유지
- 군더더기 제거 강함 (특히 D 카테고리 AI 관용구)
- 이모지/볼드/불릿 금지 (코드 주석이므로)

## 에러 / 엣지 케이스

- 입력이 비어있거나 100자 미만: "윤문할 텍스트가 너무 짧음. 50자 이상 prose 필요"
- 한국어가 아님 (한글 비율 30% 미만): 사용자에게 "이 텍스트는 한국어 비율이 낮음. 진행할까?" 확인
- 코드 블록만 있고 prose 없음: "처리할 prose 없음"
- detections 비어있음 (감지된 AI tell 0개): Step 2~4 건너뛰고 "감지된 AI tell 없음. 원문 그대로." 보고
- Agent 호출 실패: 1회 재시도, 그래도 실패면 사용자에게 ROLLBACK + 에러 메시지

## 톤 가이드 (Rewriter용 메타 지침)

사용자의 한국어 스타일은:
- 짧고 직설적
- 군더더기 없음
- 격식체와 반말 혼용 (메시지 톤 따라)
- 영어 기술 용어 그대로 (PR, Slack, Jira, K8s 등 — 한국어로 번역하지 마라)
- 감정 어휘 절제 ("정말 좋은", "매우 중요한" 같은 강조 부사 거의 안 씀)

수정안 만들 때 이 스타일에 맞춰라. AI 산출물이 "~을 통해", "~에 대해", "~라는 점에서" 같이 길게 늘어뜨리는 걸 짧게 압축하는 방향.

## Codex 호환

이 스킬을 Codex CLI에서도 쓰려면:
```bash
mkdir -p ~/.codex/skills
ln -s ~/.claude/skills/humanize-kr ~/.codex/skills/humanize-kr
```
Codex가 description 매칭으로 자동 발동 가능. 명시 호출은 `$humanize-kr`.
