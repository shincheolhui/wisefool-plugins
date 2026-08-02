# work-lifecycle

Makes Claude Code work like a disciplined teammate — triage, isolated worktree, tracked issue, verified merge — and keeps what each task taught you in the place you chose, asking once instead of deciding for you. — Claude Code 를 규율 있는 팀 동료처럼 — 질문엔 조사만, 착수엔 격리 워크트리→이슈 추적→검증된 머지. 작업이 남긴 지식은 사용자가 정한 자리에 남기고, 정해지지 않았으면 대신 정하지 않고 한 번 묻습니다.

[English](#english) | [한국어](#한국어)

---

## English

### Table of contents

1. [Why this exists](#why-this-exists)
2. [How it works (overview)](#how-it-works-overview)
3. [Installation](#installation)
4. [Usage](#usage)
5. [Project adapter reference](#project-adapter-reference)
6. [Example sessions](#example-sessions)
7. [Behavior details and edge cases](#behavior-details-and-edge-cases)
8. [Security](#security)
9. [Troubleshooting](#troubleshooting)
10. [Update / Uninstall](#update--uninstall)

### Why this exists

Working with an AI coding agent without a fixed procedure tends to fail in the same ways:

- Work happens directly on `main` and collides with other sessions or teammates
- "Done" gets declared on mock/unit tests alone — and the feature doesn't actually work
- Commits carry no issue reference, so months later nobody knows why a change was made
- Every session improvises a different process, so quality depends on the day
- Bugs and ideas discovered mid-task get buried in the conversation and lost
- What a task taught you — a decision, an incident, a dead end — lives only in the conversation, so the next session starts blind

This skill turns each of those into a guarantee, one for one:

- An **isolated worktree per task** — `main` stays clean, sessions never collide
- A **real end-to-end verification gate** — mock-only sign-off is forbidden
- **Issue-keyed** branches, commits and merge titles — `git log --oneline` reads as an issue index
- The **same before / during / after procedure** in every session
- Side-findings are **captured to the backlog immediately**, without derailing the current task
- Records land **where you decided they belong** — and when you have not decided, the skill **asks once** instead of choosing for you

### How it works (overview)

Every turn starts with **triage**: questions get investigation and a report only (no files touched); an explicit kickoff signal ("go ahead", "implement it") enters the work procedure; tiny fixes (typos, a few lines) may skip the ceremony.

Once work starts: **before** (fetch → worktree + branch → adapter check → plan sized to the task → tracker issue → branch renamed with the issue key) → **during** (meaningful-unit commits, own-files-only staging, push every commit, decisions logged as issue comments) → **after** (test gate → real e2e verification → docs sync → user-approved `--no-ff` merge → issue closed → an incident record if damage actually occurred → worktree removed, branch kept).

> **The canonical procedure is [SKILL.md](skills/work-lifecycle/SKILL.md).** It is a human-readable markdown file and it is *exactly* what the model follows — the spec and the behavior are the same document. This README intentionally does not restate it; when in doubt about any rule, SKILL.md is the authority.

### Installation

**Step 1 — add the marketplace.** Inside Claude Code:

```
/plugin marketplace add shincheolhui/wisefool-plugins
```

**Step 2 — install the plugin:**

```
/plugin install work-lifecycle@wisefool-plugins
```

**Step 3 — restart the Claude Code session.** Plugins are loaded at session start; the current session does not see a plugin installed mid-session.

**Step 4 — verify the installation.** Any of the following confirms it:

- Run `/plugin` → *Manage plugins*: `work-lifecycle` should be listed with status **enabled**.
- From a terminal: `claude plugin list` should include `work-lifecycle@wisefool-plugins`.
- Functional check: ask a question ("why is X failing?") — the skill must **not** trigger any file changes; then say "go ahead and fix it" — the session should start the *before* steps (fetch, worktree, plan) instead of editing files immediately.

**Prerequisites** (what each one is for):

| Requirement | Needed for | If missing |
|---|---|---|
| Claude Code with plugin support (`/plugin` command available) | Everything | Update Claude Code |
| Git **2.5+** (`git worktree` support) | Worktree isolation | Update git; the plugin itself is pure markdown, any OS works |
| A remote (`origin`) you can push to | Push-every-commit, remote backup | Push steps are skipped in a local-only repository; everything else still applies |
| *(Optional)* Issue tracker reachable from Claude Code | Issue create / comment / close steps | Falls back to session todo lists — see [adapter reference](#project-adapter-reference) |
| *(Optional)* A reachable place to keep records — a repository path, a wiki, a Confluence space | Plan documents, incident records | Records fall back to the issue. **A location you declare but Claude Code cannot reach would make the rule fail silently** — so the skill checks a declared location at kickoff and tells you when it cannot get through, rather than quietly writing somewhere else |
| *(Optional)* Planning skills (e.g. superpowers) | Spec/plan documents for large tasks | Specs and plans are written as plain markdown documents |

### Usage

There is no command to run in daily use — the skill activates on its own:

| You say | What happens |
|---|---|
| A question — "why does login fail?", "is this possible?" | Investigation and a report. **No files are touched.** Even if a fix is obvious, it is not applied until you say so |
| A kickoff signal — "go ahead", "implement it", "fix it" | The *before* steps run: fetch → worktree + branch → adapter check (an undecided location is asked about once; a declared one is confirmed reachable) → plan sized to the task → tracker issue created and moved to In Progress → branch renamed with the issue key. Then implementation proceeds under the *during* rules |
| A finish signal — "merge it", "wrap up" | The *after* steps run: test gate → real e2e verification → docs sync → merge (asks for your approval first) → issue closed → an incident record if damage actually occurred → worktree removed, branch kept |

Signals are recognized in **any language**. To invoke the skill explicitly: `/work-lifecycle:work-lifecycle`.

**What you will see in your repository:**

- Worktrees under `.claude/worktrees/` (default location) — created per task, removed after merge
- Branches named `<prefix>/<ISSUE-KEY>-<slug>`, e.g. `fix/ABC-12-login-timeout`; prefixes: `feature/` `fix/` `refactor/` `docs/` `chore/`
- Commit subjects ending with the issue key, e.g. `fix: handle login timeout (ABC-12)`
- Merge commits titled `Merge: <ISSUE-KEY> <summary>` — so `git log --oneline` on main doubles as an issue index
- Merged branches are **kept** (locally and on the remote) as history and rollback points

### Project adapter reference

The skill reads project-specific values from the **`CLAUDE.md` of the project it is running in**. Every key is optional — a safe default applies when a key is absent. Write them in plain prose or bullets; there is no rigid syntax, but each value must be unambiguous (exact commands, exact keys).

**Three states, not two.** A key is *declared* (the skill follows it), *delegated* (you wrote something like `Knowledge locations: delegate` — the skill decides and tells you what it chose), or *undecided* (nothing written). For most keys, undecided just means the default. But for the two keys that decide **where your issues and knowledge live**, undecided is not taken as permission to choose for you — the skill asks once, at the moment the answer is first needed, and records your answer so it never asks again.

That means you can run this plugin three ways, and you pick:

| You want | Write this |
|---|---|
| Full control | Declare the locations up front |
| To be asked, then never again | Write nothing — answer the one question when it comes |
| The skill to handle it silently | Declare delegation once (it still reports where things went) |

A preference that is really about *you* rather than one project — "always just decide for me" — goes once into your global `~/.claude/CLAUDE.md` and applies everywhere; a project's own declaration overrides it.

**What the skill may write.** Answering a bootstrap question lets the skill add that answer to your `CLAUDE.md`. It may record **only what you actually answered** — never a value it inferred — it keeps that edit as **its own commit** so you can see it, and it **reports where records went even when you delegated**. "Decide it for me" is not "do it behind my back".

#### 1. Issue tracker

| | |
|---|---|
| **What to write** | Tracker type, project key, and how to change status (transition names or IDs) |
| **Example** | `Issue tracker: Jira, project key ABC (transitions: In Progress=21, Done=41)` |
| **Used at** | Issue creation and In-Progress transition (before) · decision/blocker comments (during) · result comment and Done transition (after) |
| **Additional requirement** | The tracker must actually be reachable from Claude Code — e.g. the Atlassian plugin/MCP for Jira, the GitHub plugin/MCP for GitHub Issues — and authenticated (`/mcp`). **The skill confirms this itself** with one real read at kickoff, and tells you if it cannot get through |
| **Default when absent** | Undecided is not a default: the skill asks once, the first time an issue would be created, and records your answer. Only once you delegate — or say you want no tracker — do all issue steps become the session todo list, with branch names carrying no issue key (`fix/<slug>` form) |

#### 2. Test gate

| | |
|---|---|
| **What to write** | The exact command(s) that must pass, and when they are mandatory |
| **Example** | ``Test gate: `npm test` — run after every commit that touches src/, and before every merge`` |
| **Used at** | After domain-code commits (during) and before merge (after). A failing gate means the offending commit is reverted and reworked — it is never merged over |
| **Default when absent** | The project's standard test runner is run once before merge, if one exists |

#### 3. Docs gate

| | |
|---|---|
| **What to write** | Which documents must be updated when behavior changes, and any doc-check command |
| **Example** | `Docs gate: update the matching page under docs/ when an API or UI changes; run scripts/check_docs.py` |
| **Used at** | The docs-sync step before merge (after) |
| **Default when absent** | Only user-facing documentation of the changed feature is checked |

#### 4. Knowledge locations

| | |
|---|---|
| **What to write** | Where each kind of record is kept. A location may be a repository path, an external space (wiki, Confluence), or the tracker itself — issue management and knowledge keeping do not have to live in the same system |
| **Example** | `Knowledge locations: plans → docs/plans/; incident records → Confluence "Troubleshooting" space; design decisions → issue comments` |
| **Used at** | Planning (before) for spec/plan documents · the incident-record step (after) · lesson records (after) |
| **Additional requirement** | Same as the tracker: the location must actually be reachable from Claude Code. A destination that is declared but cannot be reached would make the rule **fail silently** — the worst outcome — so the skill checks it at kickoff instead of at writing time, and an unreachable location is **reported to you, never quietly substituted**. It also finishes each record with a reference it can point to (an issue URL, a page URL, a committed path); if it cannot produce one, it says the record did not happen rather than reporting success |
| **Default when absent** | Undecided is not a default: the skill asks once, the first time such a record is needed, and records your answer. Only once you delegate does it fall back to the issue (description or comment), and to the todo list when there is no tracker. It will **never** create a directory in your project |
| **Compatibility** | `Plan location: <path>` (introduced in 1.2.0) is still accepted as the plan entry |

Choosing a location is usually decided by two things: **audience** — developer-only knowledge belongs in the repository, next to the code and subject to review, while anything non-developers must read belongs in a wiki or Confluence; and **versioning** — knowledge that must rewind together with the code belongs in the repository, whereas a point-in-time record does not.

**When not to declare one.** If your tracker lives in the same place as the code and its issues are permanent (GitHub Issues on the same repository, say), leaving the plan entry undeclared is often the better choice: the plan stays in the issue, permanently linked from every commit that carries the key, and the repository does not accumulate process documents that go stale the moment the work merges. This is exactly what this repository does — see [#3](https://github.com/shincheolhui/wisefool-plugins/issues/3).

#### 5. Commit convention

| | |
|---|---|
| **What to write** | Language, subject format, prefixes, body style |
| **Example** | `Commit convention: Korean, subject ≤ 50 chars with fix:/docs:/refactor: prefix, bullet body, issue key at the end of the subject` |
| **Used at** | Every commit (during) and the merge commit title (after) |
| **Default when absent** | Conventional Commits (`fix:`, `feat:`, …) with the issue key appended to the subject |

#### 6. Domain invariants

| | |
|---|---|
| **What to write** | Rules the model must never violate on its own, even if they seem like improvements |
| **Example** | `Domain invariants: never add trading guards/filters the user did not request; backtest and live logic must change together` |
| **Used at** | Every implementation decision (during). When the model believes a violation is genuinely needed, it must **propose first and get consent** — never apply silently |
| **Default when absent** | None |

**Complete copy-paste example** for a project `CLAUDE.md`:

```markdown
## Work workflow (work-lifecycle adapter)
- Issue tracker: Jira, project key ABC (transitions: In Progress=21, Done=41)
- Test gate: `npm test` — run before every merge
- Docs gate: update the relevant page under docs/ when behavior changes
- Knowledge locations: plans → docs/plans/; incident records → Confluence "Troubleshooting" space
- Commit convention: Conventional Commits, subject ≤ 50 chars, issue key at the end
- Domain invariants: never change the pricing formula without explicit approval
```

**Precedence:** if your project has its own canonical procedure document, state it in `CLAUDE.md` (e.g. *"The canonical work procedure is docs/ops/work-lifecycle.md"*). That document then **takes precedence** over this skill — the skill defers to it instead of applying its own procedure.

### Example sessions

**Scenario A — question only (nothing is modified):**

```text
You:    Why does login keep failing?
Claude: (reads code, reproduces, reports the root cause and a fix proposal)
        — no worktree, no branch, no file changes. It waits for your decision.
```

**Scenario B — small fix, tracker connected:**

```text
You:    Fix it.
Claude: git fetch → worktree + branch fix/login-timeout
        → plan (small task → todo list)
        → creates issue ABC-12 in the tracker, moves it to In Progress
        → renames the branch to fix/ABC-12-login-timeout
        → implements; commits "fix: … (ABC-12)"; pushes every commit

You:    Merge it.
Claude: runs the test gate → performs one real login to verify end to end
        → updates the docs → asks for merge approval
        → merge --no-ff "Merge: ABC-12 fix login timeout" → push
        → closes ABC-12 with a result comment
        → removes the worktree, keeps the branch
```

**Scenario C — large feature (design needed):**

```text
You:    Build a notification system.
Claude: judges the task "large" → creates the issue at kickoff
        (long planning must not be lost if the session ends)
        → brainstorming → spec document → implementation plan
        → updates the issue description with the confirmed scope
        → implements plan step by step, one meaningful commit each
        If planning splits the work into independent tasks: only the first
        becomes the In-Progress issue; the rest are filed as backlog issues.
```

**Scenario D — nothing declared in the adapter:**

```text
The first time an issue would be created, Claude stops and asks —
it does not pick for you:

  "No issue tracker is declared. Use GitHub Issues (my suggestion) /
   another tracker / skip issues and use a todo list from now on?
   I'll record your answer in CLAUDE.md as its own commit."

Answer once and it never asks again.

If you choose the todo list (or delegate): same flow as B, except
issue steps become session todo items and the branch is
fix/login-timeout (no issue key). Everything else — worktree,
staging, push, gates, merge format — is identical.
```

### Behavior details and edge cases

- **Tiny-fix exception.** Typos, comments, a few obvious lines: the ceremony (issue, plan) is skipped. Committing directly on `main` is allowed only when working alone with no conflict risk; otherwise a worktree is still used.
- **Ambiguous task size** is always rounded **up** (e.g. "small or medium?" → medium).
- **Merge requires your approval.** The skill never merges to `main` silently; it asks after the gates pass.
- **Merge conflicts** are resolved by merging `main` into the work branch first (reverse merge), then retrying — **never by rebase** (shared history is preserved).
- **Concurrent sessions.** Each session touches only its own worktree and branch. Staging is always per-file (`git add <path>`), never `git add .`, so parallel sessions in one repository do not contaminate each other's commits.
- **Branch accumulation** is by design — merged branches are rollback points. If they pile up, choose your own pruning policy (e.g. delete after N months); the skill will not delete them for you.
- **Mid-work side-findings** (unrelated bugs, ideas) are filed to the tracker backlog immediately and work resumes; they do not expand the current task's scope.
- **Local-only repository** (no `origin`): push steps are skipped; all other rules apply unchanged.

### Security

This plugin contains **no executable code** — no hooks, no scripts, no MCP servers. It is a single markdown skill file that only adds instructions to the model. Installing it cannot run anything on your machine.

### Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Skill does not activate right after install | Plugins load at session start — restart the Claude Code session |
| Skill not listed in `/plugin` | Check the marketplace was added (`/plugin marketplace list`), then reinstall |
| Skill activates when you only wanted an answer | Phrase it as a question ("what do you think about…?") — questions never trigger work. If it still starts, say "question only" |
| Issue steps do nothing | You delegated the tracker or told the skill you want none — it only falls back silently after you have said so. Declare a reachable tracker in the `CLAUDE.md` adapter (e.g. Atlassian/GitHub plugin + `/mcp` auth) to switch them back on |
| You are asked where to keep issues or records, and you would rather not be | That question is asked once per key, only when the answer is first needed. To switch it off for good, declare delegation (`Knowledge locations: delegate`) — in the project's `CLAUDE.md`, or once in your global `~/.claude/CLAUDE.md` for every project |
| You declared a location (a Confluence space, say) but no record ever appears there | The skill checks a declared location at kickoff and reports what it could not reach — if you saw no such report, the destination was reached and the record carries a reference in the issue's result comment. If you did see one, connect the plugin/MCP for that destination and authenticate (`/mcp`); the skill will not write somewhere else on its own |
| Project already has its own workflow skill or procedure document | They coexist; declare the canonical document in `CLAUDE.md` and it takes precedence over this skill |
| Behavior seems to ignore your adapter values | The adapter is read from the `CLAUDE.md` of the project being worked on — check the values are there and unambiguous (exact commands, exact keys) |
| Want to pause the skill temporarily | `/plugin disable work-lifecycle`, re-enable with `/plugin enable work-lifecycle` |

### Update / Uninstall

Update (marketplace catalog, then the plugin):

```
/plugin marketplace update wisefool-plugins
/plugin update work-lifecycle
```

Uninstall:

```
/plugin uninstall work-lifecycle
```

Changelog: [CHANGELOG.md](CHANGELOG.md)

---

## 한국어

### 목차

1. [왜 필요한가](#왜-필요한가)
2. [동작 개요](#동작-개요)
3. [설치](#설치)
4. [사용법](#사용법)
5. [프로젝트 어댑터 레퍼런스](#프로젝트-어댑터-레퍼런스)
6. [사용 시나리오](#사용-시나리오)
7. [동작 세부와 엣지 케이스](#동작-세부와-엣지-케이스)
8. [보안](#보안)
9. [문제 해결](#문제-해결)
10. [업데이트 / 제거](#업데이트--제거)

### 왜 필요한가

절차 없이 AI 코딩 에이전트와 작업하면 같은 방식으로 실패하곤 합니다:

- `main` 에서 직접 작업 → 다른 세션·동료의 작업과 뒤섞임
- mock·단위 테스트만 통과하고 "완료" 선언 → 실제로는 동작하지 않음
- 커밋에 이슈 연결이 없음 → 몇 달 뒤 왜 바꿨는지 아무도 모름
- 세션마다 절차를 즉흥으로 만듦 → 품질이 그날그날 복불복
- 작업 중 발견한 버그·아이디어가 대화 속에 묻혀 유실
- 작업이 남긴 것(결정·사고·막다른 길)이 대화에만 있어서, 다음 세션은 아무것도 모른 채 시작

이 skill 은 그 각각을 1:1 로 보장으로 바꿉니다:

- 작업마다 **격리된 워크트리** — `main` 은 항상 깨끗, 세션 간 충돌 없음
- **실제 실행 e2e 검증 게이트** — mock 만으로 완료 선언 금지
- 브랜치·커밋·머지 제목에 **이슈 키** — `git log --oneline` 이 이슈 목차가 됨
- 모든 세션이 **동일한 전·중·후 절차**
- 곁가지는 **즉시 백로그로 캐처** — 현재 작업 흐름을 끊지 않음
- 기록은 **사용자가 정한 자리**에 남고, 정해지지 않았으면 대신 고르지 않고 **한 번 묻습니다**

### 동작 개요

모든 턴은 **분류**로 시작합니다: 질문은 조사·보고만(파일 무변경), 명시적 착수 신호("진행합시다", "구현해주세요")에만 작업 절차 진입, 잔손질(오탈자·수 줄)은 절차 생략 가능.

작업이 시작되면: **작업 전**(fetch → 워크트리+브랜치 → 어댑터 확인 → 규모별 계획 → 트래커 이슈 → 이슈 키로 브랜치 rename) → **작업 중**(의미 단위 커밋, 내 파일만 스테이징, 매 커밋 push, 결정 사항 이슈 댓글) → **작업 후**(테스트 게이트 → 실제 e2e 검증 → 문서 동기화 → 사용자 승인 후 `--no-ff` 머지 → 이슈 완료 → 실제 피해가 있었던 작업이면 사고 기록 → 워크트리 제거·브랜치 보존).

> **절차의 정본은 [SKILL.md](skills/work-lifecycle/SKILL.md) 입니다** (본문은 영어). 사람이 읽을 수 있는 마크다운이면서 모델이 따르는 것 *그 자체*라, 명세와 동작이 같은 문서입니다. 이 README 는 절차를 중복 서술하지 않습니다 — 규칙이 궁금하면 SKILL.md 가 정답입니다.

### 설치

**1단계 — 마켓플레이스 추가.** Claude Code 안에서:

```
/plugin marketplace add shincheolhui/wisefool-plugins
```

**2단계 — 플러그인 설치:**

```
/plugin install work-lifecycle@wisefool-plugins
```

**3단계 — Claude Code 세션 재시작.** 플러그인은 세션 시작 시 로드되므로, 설치한 그 세션에서는 보이지 않습니다.

**4단계 — 설치 확인.** 아래 중 아무거나:

- `/plugin` → *Manage plugins* 에 `work-lifecycle` 이 **enabled** 상태로 표시
- 터미널에서 `claude plugin list` 에 `work-lifecycle@wisefool-plugins` 포함
- 기능 확인: 질문("X 가 왜 실패하죠?")을 던지면 파일 변경이 **없어야** 하고, "고쳐주세요"라고 하면 파일을 바로 고치는 대신 *작업 전* 절차(fetch·워크트리·계획)부터 시작해야 합니다

**전제조건** (각각이 왜 필요한지):

| 요구사항 | 필요한 이유 | 없으면 |
|---|---|---|
| 플러그인을 지원하는 Claude Code (`/plugin` 명령 사용 가능) | 전체 | Claude Code 업데이트 |
| Git **2.5 이상** (`git worktree` 지원) | 워크트리 격리 | git 업데이트. 플러그인 자체는 마크다운뿐이라 OS 무관 |
| push 가능한 원격(`origin`) | 매 커밋 push·원격 백업 | 로컬 전용 저장소면 push 단계만 생략, 나머지 동일 |
| *(선택)* Claude Code 에서 접근 가능한 이슈 트래커 | 이슈 생성·댓글·완료 단계 | 세션 Todo 목록으로 대체 — [어댑터 레퍼런스](#프로젝트-어댑터-레퍼런스) 참조 |
| *(선택)* 기록을 둘 접근 가능한 자리 — 저장소 경로·위키·Confluence 스페이스 | 계획 문서, 사고 기록 | 기록이 이슈로 폴백됩니다. **선언했는데 Claude Code 가 도달하지 못하면 규칙이 조용히 실패하므로**, skill 이 착수 시점에 선언된 위치를 확인하고 닿지 못하면 알려 줍니다 — 말없이 다른 곳에 쓰지 않습니다 |
| *(선택)* 계획 보조 skill (superpowers 등) | 대형 작업의 spec/plan | 일반 마크다운 문서로 작성 |

### 사용법

일상 사용에서 실행할 명령은 없습니다 — skill 이 스스로 활성화됩니다:

| 사용자가 말하면 | 일어나는 일 |
|---|---|
| 질문 — "로그인이 왜 실패하죠?", "가능해요?" | 조사와 보고. **파일을 건드리지 않습니다.** 수정안이 자명해도 지시 전에는 적용하지 않습니다 |
| 착수 신호 — "진행합시다", "구현해주세요", "고쳐주세요" | *작업 전* 절차 실행: fetch → 워크트리+브랜치 → 어댑터 확인(미정 위치는 한 번 묻고, 선언된 위치는 도달 가능한지 확인) → 규모별 계획 → 트래커 이슈 생성·진행 중 전환 → 이슈 키로 브랜치 rename. 이후 *작업 중* 규율로 구현 |
| 마무리 신호 — "머지", "완료 처리" | *작업 후* 절차 실행: 테스트 게이트 → 실제 e2e 검증 → 문서 동기화 → 머지(먼저 승인을 요청) → 이슈 완료 → 실제 피해가 있었던 작업이면 사고 기록 → 워크트리 제거·브랜치 보존 |

신호는 **언어 불문** 인식됩니다. 명시적으로 부르려면: `/work-lifecycle:work-lifecycle`.

**저장소에서 보게 되는 것:**

- `.claude/worktrees/` 하위의 워크트리 (기본 위치) — 작업마다 생성, 머지 후 제거
- `<접두사>/<이슈키>-<슬러그>` 형식 브랜치, 예: `fix/ABC-12-login-timeout`; 접두사: `feature/` `fix/` `refactor/` `docs/` `chore/`
- 제목 끝에 이슈 키가 붙은 커밋, 예: `fix: 로그인 타임아웃 처리 (ABC-12)`
- `Merge: <이슈키> <요약>` 형식의 머지 커밋 — main 의 `git log --oneline` 이 이슈 목차가 됩니다
- 머지된 브랜치는 로컬·원격 모두 **보존** (이력·롤백 지점)

### 프로젝트 어댑터 레퍼런스

skill 은 **작업 중인 프로젝트의 `CLAUDE.md`** 에서 프로젝트별 값을 읽습니다. 모든 키는 선택 사항이고, 없으면 안전한 기본값으로 동작합니다. 형식 제약은 없지만(산문·불릿 무방) 값 자체는 모호하지 않아야 합니다 (정확한 명령어, 정확한 키).

**상태는 둘이 아니라 셋입니다.** 키는 *선언됨*(skill 이 그대로 따름), *위임됨*(`지식 기록 위치: 위임` 처럼 적어 둔 경우 — skill 이 정하되 어디로 정했는지 알려 줌), *미정*(아무것도 안 적음) 중 하나입니다. 대부분의 키는 미정이면 그냥 기본값입니다. 하지만 **당신의 이슈와 지식이 어디에 쌓일지**를 정하는 두 키는, 미정을 "대신 정해도 좋다"로 받아들이지 않습니다 — skill 이 그 답이 처음 필요해지는 순간에 **한 번만** 묻고, 답을 기록해 다시는 묻지 않습니다.

즉 이 플러그인은 세 가지 방식으로 쓸 수 있고, 선택은 사용자 몫입니다:

| 원하는 것 | 이렇게 적습니다 |
|---|---|
| 완전한 통제 | 위치를 미리 선언 |
| 한 번 묻고 그 뒤로는 안 묻기 | 아무것도 안 적음 — 질문이 오면 그때 답변 |
| skill 이 알아서 처리 | 위임을 한 번 선언 (그래도 어디에 넣었는지는 보고합니다) |

특정 프로젝트가 아니라 *사람* 단위 선호라면("나는 항상 알아서 해주면 좋겠다") 전역 `~/.claude/CLAUDE.md` 에 한 번 적으면 모든 프로젝트에 적용되고, 프로젝트 선언이 그것을 덮어씁니다.

**skill 이 쓸 수 있는 것.** 부트스트랩 질문에 답하면 skill 이 그 답을 `CLAUDE.md` 에 추가합니다. 이때 **사용자가 실제로 답한 것만** 기록하며(추론한 값은 절대 적지 않습니다), 그 편집을 **별도 커밋**으로 분리해 눈에 보이게 하고, **위임한 경우에도 기록이 어디로 갔는지 보고합니다.** "알아서 해줘"가 "몰래 해줘"는 아니니까요.

#### 1. 이슈 트래커

| | |
|---|---|
| **적을 것** | 트래커 종류, 프로젝트 키, 상태 전환 방법(전환 이름 또는 ID) |
| **예시** | `이슈 트래커: Jira, 프로젝트 키 ABC (전환: 진행 중=21, 완료=41)` |
| **쓰이는 시점** | 이슈 생성·진행 중 전환(작업 전) · 결정/막힘 댓글(작업 중) · 결과 댓글·완료 전환(작업 후) |
| **추가 전제** | 트래커가 Claude Code 에서 실제로 접근 가능해야 합니다 — Jira 는 Atlassian 플러그인/MCP, GitHub Issues 는 GitHub 플러그인/MCP — 그리고 인증(`/mcp`) 완료. **이 확인은 skill 이 직접 합니다** — 착수 시점에 실제 읽기 1회로 확인하고, 닿지 못하면 알려 줍니다 |
| **없으면** | 미정은 기본값이 아닙니다. 이슈를 처음 만들어야 하는 순간에 skill 이 한 번 묻고 답을 기록합니다. 위임하거나 트래커를 쓰지 않겠다고 답한 뒤에야 이슈 단계가 세션 Todo 목록으로 대체되고, 브랜치명이 키 없이 `fix/<슬러그>` 형식이 됩니다 |

#### 2. 테스트 게이트

| | |
|---|---|
| **적을 것** | 반드시 통과해야 하는 정확한 명령어와, 언제 필수인지 |
| **예시** | ``테스트 게이트: `npm test` — src/ 를 건드린 매 커밋 후, 그리고 머지 전 필수`` |
| **쓰이는 시점** | 도메인 코드 커밋 후(작업 중), 머지 전(작업 후). 게이트 실패 시 해당 커밋을 revert 하고 재작업 — 실패 상태로 머지하지 않습니다 |
| **없으면** | 프로젝트 표준 테스트 러너가 있으면 머지 전 1회 실행 |

#### 3. 문서 게이트

| | |
|---|---|
| **적을 것** | 동작 변경 시 갱신해야 할 문서 대상과 점검 명령(있으면) |
| **예시** | `문서 게이트: API·UI 변경 시 docs/ 하위 해당 페이지 갱신, scripts/check_docs.py 실행` |
| **쓰이는 시점** | 머지 전 문서 동기화 단계(작업 후) |
| **없으면** | 변경된 기능의 사용자 문서만 확인 |

#### 4. 지식 기록 위치

| | |
|---|---|
| **적을 것** | 기록 종류별로 어디에 남길지. 소재는 저장소 경로일 수도, 외부 공간(위키·Confluence)일 수도, 트래커 자신일 수도 있습니다 — 이슈를 관리하는 지점과 지식을 쌓는 지점이 같은 시스템일 필요는 없습니다 |
| **예시** | `지식 기록 위치: 계획 → docs/plans/; 사고 기록 → Confluence "트러블슈팅" 스페이스; 설계 결정 → 이슈 댓글` |
| **쓰이는 시점** | 계획 수립(작업 전)의 spec·plan 문서 · 사고 기록 단계(작업 후) · 교훈 기록(작업 후) |
| **추가 전제** | 트래커와 동일 — 선언한 위치에 Claude Code 가 실제로 접근할 수 있어야 합니다. 선언했는데 도달하지 못하면 규칙이 **조용히 실패**하고 이게 최악이므로, skill 은 쓰는 순간이 아니라 착수 시점에 확인하고 **도달 실패를 사용자에게 알립니다 — 조용히 다른 곳으로 대체하지 않습니다.** 또한 기록은 가리킬 수 있는 참조(이슈 URL·페이지 URL·커밋된 경로)가 있어야 끝난 것으로 보며, 내놓지 못하면 성공 대신 기록이 일어나지 않았다고 보고합니다 |
| **없으면** | 미정은 기본값이 아닙니다. 그 기록이 처음 필요해지는 순간에 skill 이 한 번 묻고 답을 기록합니다. 위임한 뒤에야 이슈(설명 또는 댓글)로 폴백하고, 트래커도 없으면 Todo 목록으로 갑니다. 프로젝트에 디렉토리를 **새로 만드는 일은 없습니다** |
| **호환** | 1.2.0 에서 도입한 `계획 문서 위치: <경로>` 표기도 계획 항목 선언으로 계속 인정됩니다 |

위치 선택은 대개 두 가지가 결정합니다. **청중** — 개발자만 읽는 지식은 저장소(코드 옆, 리뷰에 걸림), 비개발자도 읽어야 하는 지식은 위키·Confluence. **버전 관리 필요성** — 코드와 함께 되감겨야 하는 지식은 저장소, 시점 스냅샷이면 되는 지식은 밖.

**선언하지 않는 편이 나은 경우.** 트래커가 코드와 같은 곳에 있고 이슈가 영구적이라면(같은 저장소의 GitHub Issues 등), 계획 항목은 선언하지 않는 쪽이 나을 때가 많습니다. 계획이 이슈에 남아 키를 단 모든 커밋에서 영구 링크되고, 저장소에는 머지되는 순간 낡아버릴 과정 문서가 쌓이지 않습니다. 이 저장소가 정확히 그렇게 하고 있습니다 — [#3](https://github.com/shincheolhui/wisefool-plugins/issues/3) 참조.

#### 5. 커밋 컨벤션

| | |
|---|---|
| **적을 것** | 언어, 제목 형식, 접두사, 본문 스타일 |
| **예시** | `커밋 컨벤션: 한글, 제목 50자 이내 + fix:/docs:/refactor: 접두사, 본문 불릿, 제목 끝에 이슈 키` |
| **쓰이는 시점** | 모든 커밋(작업 중)과 머지 커밋 제목(작업 후) |
| **없으면** | Conventional Commits (`fix:`, `feat:`, …) + 제목 끝 이슈 키 |

#### 6. 도메인 불변

| | |
|---|---|
| **적을 것** | 개선처럼 보여도 모델이 스스로 어겨서는 안 되는 규칙 |
| **예시** | `도메인 불변: 사용자가 요청하지 않은 매매 가드/필터 추가 금지, 백테스트와 실전 로직은 항상 함께 수정` |
| **쓰이는 시점** | 모든 구현 결정(작업 중). 위반이 정말 필요하다고 판단되면 **먼저 제안하고 동의를 받습니다** — 조용히 적용하지 않습니다 |
| **없으면** | 없음 |

**복사해 쓰는 전체 예시** (프로젝트 `CLAUDE.md` 에):

```markdown
## 작업 워크플로 (work-lifecycle 어댑터)
- 이슈 트래커: Jira, 프로젝트 키 ABC (전환: 진행 중=21, 완료=41)
- 테스트 게이트: `npm test` — 머지 전 필수
- 문서 게이트: 동작 변경 시 docs/ 하위 해당 문서 갱신
- 지식 기록 위치: 계획 → docs/plans/; 사고 기록 → Confluence "트러블슈팅" 스페이스
- 커밋 컨벤션: Conventional Commits, 제목 50자 이내, 제목 끝에 이슈 키
- 도메인 불변: 가격 계산식은 명시적 승인 없이 변경 금지
```

**우선순위:** 프로젝트에 자체 절차 정본 문서가 있으면 `CLAUDE.md` 에 그 사실을 명시하세요 (예: *"작업 절차 정본은 docs/ops/work-lifecycle.md"*). 그러면 그 문서가 이 skill 보다 **우선**하며, skill 은 자기 절차 대신 그 문서를 따릅니다.

### 사용 시나리오

**시나리오 A — 질문만 (아무것도 변경되지 않음):**

```text
사용자:  로그인이 왜 계속 실패하죠?
Claude:  (코드 확인·재현 후 원인과 수정안 보고)
         — 워크트리 없음, 브랜치 없음, 파일 무변경. 사용자 결정을 기다립니다.
```

**시나리오 B — 소형 수정, 트래커 연결 상태:**

```text
사용자:  고쳐주세요.
Claude:  git fetch → 워크트리+브랜치 fix/login-timeout
         → 계획 수립 (소형 → Todo 목록)
         → 트래커에 이슈 ABC-12 생성, "진행 중" 전환
         → 브랜치를 fix/ABC-12-login-timeout 으로 rename
         → 구현 — "fix: … (ABC-12)" 의미 단위 커밋 + 매 커밋 push

사용자:  머지해주세요.
Claude:  테스트 게이트 실행 → 실제 로그인 1회로 e2e 검증
         → 문서 갱신 → 머지 승인 요청
         → merge --no-ff "Merge: ABC-12 로그인 타임아웃 수정" → push
         → ABC-12 에 결과 댓글 + 완료 전환
         → 워크트리 제거, 브랜치 보존
```

**시나리오 C — 대형 기능 (설계가 필요한 작업):**

```text
사용자:  알림 시스템을 만들어주세요.
Claude:  규모 판정 "대형" → 착수 시점에 이슈 먼저 생성
         (계획이 길어져 세션이 끊겨도 "진행 중" 기록이 남아야 하므로)
         → 브레인스토밍 → spec 문서 → 구현 plan
         → 확정된 범위로 이슈 설명 갱신
         → plan 단계별로 의미 단위 커밋하며 구현
         계획 결과 독립 작업 여러 개로 쪼개지면: 첫 작업만 착수 이슈로,
         나머지는 백로그 이슈로 등록됩니다.
```

**시나리오 D — 어댑터에 아무것도 선언하지 않은 경우:**

```text
이슈를 처음 만들어야 하는 순간, Claude 는 대신 정하지 않고 멈춰 묻습니다:

  "이슈 트래커가 선언되어 있지 않습니다. GitHub Issues(제 제안) /
   다른 트래커 지정 / 앞으로 이슈 없이 Todo 로 진행 — 어느 쪽일까요?
   답을 CLAUDE.md 에 별도 커밋으로 기록해 두겠습니다."

한 번 답하면 다시 묻지 않습니다.

Todo 를 택하거나 위임하면: B 와 같은 흐름이되 이슈 단계가 세션 Todo 항목으로
바뀌고 브랜치명은 fix/login-timeout (이슈 키 없음). 워크트리·스테이징·push·
게이트·머지 형식 등 나머지는 전부 동일합니다.
```

### 동작 세부와 엣지 케이스

- **잔손질 예외.** 오탈자·주석·자명한 수 줄 수정은 절차(이슈·계획)를 생략합니다. `main` 직접 커밋은 단독 작업·충돌 위험 없음일 때만 허용되고, 그 외에는 잔손질이라도 워크트리를 씁니다.
- **규모 판정이 모호하면** 항상 **한 단계 위**로 취급합니다 ("소형인가 중형인가?" → 중형).
- **머지는 사용자 승인 후에만.** 게이트를 통과해도 `main` 머지는 조용히 하지 않고 먼저 물어봅니다.
- **머지 충돌**은 작업 브랜치에 `main` 을 먼저 역머지해 해소 후 재시도합니다 — **rebase 는 하지 않습니다** (공유 이력 보존).
- **동시 세션.** 각 세션은 자기 워크트리·브랜치만 다룹니다. 스테이징은 항상 파일 단위(`git add <경로>`)이고 `git add .` 는 쓰지 않으므로, 한 저장소에서 병렬 작업해도 서로의 커밋이 오염되지 않습니다.
- **브랜치가 쌓이는 것은 의도된 동작**입니다 — 머지된 브랜치는 롤백 지점입니다. 너무 쌓이면 정리 기준(예: 머지 후 N개월)은 사용자가 정합니다. skill 이 대신 삭제하지 않습니다.
- **작업 중 곁가지**(무관한 버그·아이디어)는 즉시 트래커 백로그로 등록하고 원래 작업으로 복귀합니다. 현재 작업의 범위를 늘리지 않습니다.
- **로컬 전용 저장소** (`origin` 없음): push 단계만 생략되고 나머지 규칙은 동일합니다.

### 보안

이 플러그인에는 **실행 코드가 없습니다** — hook·스크립트·MCP 서버 없이 마크다운 skill 파일 1개뿐이며, 모델에 지침만 추가합니다. 설치해도 사용자 컴퓨터에서 아무것도 실행되지 않습니다.

### 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| 설치 직후 skill 이 활성화되지 않음 | 플러그인은 세션 시작 시 로드됩니다 — Claude Code 세션을 재시작하세요 |
| `/plugin` 목록에 skill 이 없음 | 마켓플레이스 등록 확인(`/plugin marketplace list`) 후 재설치 |
| 답변만 원했는데 skill 이 작업을 시작함 | 질문형으로 물어보세요("…에 대해 어떻게 생각해요?") — 질문은 작업을 발동하지 않습니다. 그래도 시작하면 "질문입니다"라고 말하세요 |
| 이슈 단계가 동작하지 않음 | 트래커를 위임했거나 쓰지 않겠다고 답한 상태입니다 — skill 은 그렇게 답한 뒤에야 조용히 폴백합니다. 다시 켜려면 접근 가능한 트래커를 연결(Atlassian/GitHub 플러그인 + `/mcp` 인증)하고 `CLAUDE.md` 어댑터에 선언하세요 |
| 기록 위치를 묻는 게 번거로움 | 키마다 한 번씩, 그 답이 처음 필요해질 때만 묻습니다. 영구히 끄려면 위임을 선언하세요(`지식 기록 위치: 위임`) — 프로젝트 `CLAUDE.md` 에, 또는 모든 프로젝트에 적용하려면 전역 `~/.claude/CLAUDE.md` 에 한 번 |
| 위치를 선언했는데(예: Confluence 스페이스) 거기에 아무 기록도 안 생김 | skill 은 착수 시점에 선언된 위치를 확인하고 닿지 못한 대상을 보고합니다 — 그런 보고가 없었다면 목적지에는 닿았고, 기록의 참조가 이슈 결과 댓글에 실려 있습니다. 보고를 받았다면 해당 목적지의 플러그인/MCP 를 연결하고 인증(`/mcp`)하세요. skill 이 스스로 다른 곳에 쓰지는 않습니다 |
| 프로젝트에 이미 자체 워크플로 skill·절차 문서가 있음 | 공존합니다. `CLAUDE.md` 에 정본 문서를 선언하면 그 문서가 이 skill 보다 우선합니다 |
| 어댑터 값이 무시되는 것 같음 | 어댑터는 **작업 중인 프로젝트의** `CLAUDE.md` 에서 읽습니다 — 값이 거기 있는지, 모호하지 않은지(정확한 명령어·키) 확인하세요 |
| 일시적으로 끄고 싶음 | `/plugin disable work-lifecycle`, 다시 켜려면 `/plugin enable work-lifecycle` |

### 업데이트 / 제거

업데이트 (마켓플레이스 카탈로그 → 플러그인 순):

```
/plugin marketplace update wisefool-plugins
/plugin update work-lifecycle
```

제거:

```
/plugin uninstall work-lifecycle
```

변경 이력: [CHANGELOG.md](CHANGELOG.md)

---

## License

MIT — see [LICENSE](../../LICENSE).
