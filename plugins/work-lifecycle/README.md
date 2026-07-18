# work-lifecycle

An **opinionated** Claude Code skill that enforces a before / during / after work-lifecycle procedure in every session. — 작업 전·중·후 라이프사이클 절차를 모든 세션에 강제하는 **의견이 강한(opinionated)** Claude Code skill.

[English](#english) | [한국어](#한국어)

---

## English

### Why

Working with an AI coding agent without a fixed procedure tends to fail in the same ways:

- Work happens directly on `main` and collides with other sessions or teammates
- "Done" gets declared on mock/unit tests alone — and the feature doesn't actually work
- Commits carry no issue reference, so months later nobody knows why a change was made
- Every session improvises a different process, so quality depends on the day
- Bugs and ideas discovered mid-task get buried in the conversation and lost

This skill turns each of those into a guarantee, one for one:

- An **isolated worktree per task** — `main` stays clean, sessions never collide
- A **real end-to-end verification gate** — mock-only sign-off is forbidden
- **Issue-keyed** branches, commits and merge titles — `git log --oneline` reads as an issue index
- The **same before / during / after procedure** in every session
- Side-findings are **captured to the backlog immediately**, without derailing the current task

### What it does

**Triage — every turn starts here.** Questions and investigations get report-only responses; no files are touched. Work begins only on an explicit kickoff signal ("go ahead", "implement it"). Tiny fixes (typos, a few lines) may skip the ceremony.

**Before — set up so nothing collides.** Fetch → worktree + branch off main → plan sized to the task (todo list / plan document / brainstorming→spec→plan) → create the tracker issue right after the plan is fixed — *so the issue carries an accurate title, scope and plan link, and exists before any real change happens* → rename the branch with the issue key.

**During — leave a trail.** Commit in meaningful units and stage only your own files (other sessions may share the repository). Push every commit — *backup against local loss, visibility from other machines, automatic tracker linking*. Record decisions and surprises as issue comments. Capture unrelated findings to the backlog immediately.

**After — prove it, then merge.** Run the project's test gate → verify by actually running the thing (*passing tests ≠ working software*) → sync docs → `--no-ff` merge with the issue key in the title → close the issue → remove the worktree but **keep the branch** — *it is history and a rollback point*.

### Install

```
/plugin marketplace add shincheolhui/wisefool-plugins
/plugin install work-lifecycle@wisefool-plugins
```

### Requirements

| Requirement | Needed for | Notes |
|---|---|---|
| Claude Code with plugin support | Everything | Check with `claude --version`; the `/plugin` command must be available |
| Git **2.5+** | Worktree/branch discipline | `git worktree` was introduced in 2.5. Any OS — the plugin is a single markdown file, no scripts |
| A git repository with a remote (`origin`) you can push to | "push every commit" and merge steps | In a local-only repository the push steps are skipped; the rest of the procedure still applies |
| *(Optional)* Issue tracker reachable from Claude Code | Issue creation/comments/close steps | e.g. Jira via the Atlassian plugin/MCP, GitHub Issues via the GitHub plugin/MCP — each needs its own authentication (`/mcp`). **Without one, issue steps fall back to todo lists** |
| *(Optional)* Project `CLAUDE.md` adapter section | Project-specific values | See Usage below. Safe defaults apply when undefined |
| *(Optional)* Planning skills (e.g. superpowers brainstorming/plans) | Large-scale work | Without them, specs/plans are written as plain documents |

### Usage

No command to run — the skill **activates automatically**:

- Ask a question ("why is this failing?") → investigation and a report only; no files touched.
- Give a kickoff signal ("go ahead", "implement it") → the skill walks through the *before* steps: fetch → worktree + branch → plan sized to the task → create/start the tracker issue → rename the branch with the issue key.
- Ask to finish ("merge it", "wrap up") → the *after* steps: test gate → real e2e verification → docs sync → `--no-ff` merge titled with the issue key → close the issue → remove the worktree, keep the branch.

Signals are recognized in **any language**. To invoke the skill explicitly: `/work-lifecycle:work-lifecycle`.

Worktrees are created inside your repository (by default under `.claude/worktrees/`) and removed after the merge — the **branch is always kept** as a rollback point.

The summary above is only the skeleton. The full procedure — task-sizing table (tiny/small/medium/large and what plan each requires), multi-issue branching rules, backlog capture for side-findings — is in [SKILL.md](skills/work-lifecycle/SKILL.md).

**Project adapter** — define project-specific values in your project's `CLAUDE.md` and the skill will use them (all optional; defaults apply when absent):

```markdown
## Work workflow (work-lifecycle adapter)
- Issue tracker: Jira, project key ABC (transitions: In Progress=21, Done=41)
- Test gate: `npm test` (run before every merge)
- Docs gate: update the relevant page under docs/ when behavior changes
- Commit convention: Conventional Commits, subject ≤ 50 chars, issue key at the end
- Domain invariants: never add trading guards/filters the user did not request
```

If your project has its own canonical procedure document, state it in `CLAUDE.md` — that document **takes precedence** over this skill.

### Example session

```text
You:    Why does login keep failing?
Claude: (investigates and reports findings — no files touched)

You:    Fix it.
Claude: git fetch → worktree + branch fix/login-timeout
        → plan (small task → todo list)
        → creates issue ABC-12 in the tracker, moves it to In Progress
        → renames the branch to fix/ABC-12-login-timeout
        → implements; commits "fix: … (ABC-12)"; pushes every commit

You:    Merge it.
Claude: runs the test gate → performs one real login to verify end to end
        → updates the docs → merge --no-ff "Merge: ABC-12 fix login timeout"
        → closes ABC-12 → removes the worktree, keeps the branch
```

### Security

This plugin contains **no executable code** — no hooks, no scripts, no MCP servers. It is a single markdown skill file that only adds instructions to the model.

### Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Skill does not activate right after install | Plugins load at session start — restart the Claude Code session |
| Project already has its own workflow skill or procedure document | They coexist; the project's canonical document takes precedence over this skill (state it in `CLAUDE.md`) |
| Issue steps do nothing | No issue tracker is connected — connect one (e.g. Atlassian/GitHub plugin + `/mcp` auth) or let it fall back to todo lists |
| Want to pause it temporarily | `/plugin disable work-lifecycle`, re-enable with `/plugin enable work-lifecycle` |

### Update

```
/plugin marketplace update wisefool-plugins
/plugin update work-lifecycle
```

### Uninstall

```
/plugin uninstall work-lifecycle
```

### Changelog

See [CHANGELOG.md](CHANGELOG.md).

---

## 한국어

### 왜 필요한가

절차 없이 AI 코딩 에이전트와 작업하면 같은 방식으로 실패하곤 합니다:

- `main` 에서 직접 작업 → 다른 세션·동료의 작업과 뒤섞임
- mock·단위 테스트만 통과하고 "완료" 선언 → 실제로는 동작하지 않음
- 커밋에 이슈 연결이 없음 → 몇 달 뒤 왜 바꿨는지 아무도 모름
- 세션마다 절차를 즉흥으로 만듦 → 품질이 그날그날 복불복
- 작업 중 발견한 버그·아이디어가 대화 속에 묻혀 유실

이 skill 은 그 각각을 1:1 로 보장으로 바꿉니다:

- 작업마다 **격리된 워크트리** — `main` 은 항상 깨끗, 세션 간 충돌 없음
- **실제 실행 e2e 검증 게이트** — mock 만으로 완료 선언 금지
- 브랜치·커밋·머지 제목에 **이슈 키** — `git log --oneline` 이 이슈 목차가 됨
- 모든 세션이 **동일한 전·중·후 절차**
- 곁가지는 **즉시 백로그로 캐처** — 현재 작업 흐름을 끊지 않음

### 무엇을 하는가

**분류 — 모든 턴의 시작.** 질문·조사는 보고만 하고 파일을 건드리지 않습니다. 명시적 착수 신호("진행합시다", "구현해주세요")에만 작업에 진입합니다. 잔손질(오탈자·수 줄)은 절차를 생략할 수 있습니다.

**작업 전 — 충돌하지 않게 준비.** fetch → main 기반 워크트리+브랜치 → 규모별 계획(Todo 목록 / plan 문서 / 브레인스토밍→spec→plan) → **계획 확정 직후** 트래커 이슈 생성 — *정확한 제목·범위·plan 링크를 담고, 실질 변경 전에 "진행 중" 기록을 남기기 위해* → 브랜치명에 이슈 키 rename.

**작업 중 — 흔적을 남기며.** 의미 단위 커밋, 내 작업 파일만 개별 스테이징(같은 저장소를 다른 세션이 쓸 수 있으므로). 매 커밋 push — *로컬 유실 대비 + 외부 기기 가시성 + 트래커 자동 연동*. 설계 결정·예상 밖 동작은 이슈 댓글로 기록. 무관한 발견은 즉시 백로그로.

**작업 후 — 증명하고 머지.** 프로젝트 테스트 게이트 실행 → 실제로 돌려서 검증(*테스트 통과 ≠ 실제 동작*) → 문서 동기화 → 이슈 키 제목의 `--no-ff` 머지 → 이슈 완료 → 워크트리만 제거하고 **브랜치는 보존** — *이력이자 롤백 지점이므로*.

### 설치

```
/plugin marketplace add shincheolhui/wisefool-plugins
/plugin install work-lifecycle@wisefool-plugins
```

### 요구사항

| 요구사항 | 필요한 이유 | 비고 |
|---|---|---|
| 플러그인을 지원하는 Claude Code | 전체 | `claude --version` 확인, `/plugin` 명령이 있어야 함 |
| Git **2.5 이상** | 워크트리·브랜치 규율 | `git worktree` 가 2.5에서 도입됨. OS 무관 — 플러그인은 마크다운 1장, 스크립트 없음 |
| push 가능한 원격(`origin`)이 있는 git 저장소 | "매 커밋 push"·머지 단계 | 로컬 전용 저장소면 push 단계는 건너뛰고 나머지 절차는 동일하게 적용 |
| *(선택)* Claude Code 에서 접근 가능한 이슈 트래커 | 이슈 생성·댓글·완료 단계 | 예: Jira → Atlassian 플러그인/MCP, GitHub Issues → GitHub 플러그인/MCP. 각각 별도 인증 필요(`/mcp`). **없으면 이슈 단계는 Todo 목록으로 대체** |
| *(선택)* 프로젝트 `CLAUDE.md` 어댑터 섹션 | 프로젝트별 값 적용 | 아래 사용법 참조. 미정의 시 안전한 기본값으로 동작 |
| *(선택)* 계획 보조 skill (superpowers brainstorming/plans 등) | 대형 작업 | 없으면 spec/plan 을 일반 문서로 작성 |

### 사용법

실행할 명령은 따로 없습니다 — skill 이 **자동 활성화**됩니다:

- 질문("왜 안 되죠?")을 하면 → 조사·보고만 하고 파일을 건드리지 않습니다.
- 착수 신호("진행합시다", "구현해주세요")를 주면 → *작업 전* 절차를 밟습니다: fetch → 워크트리+브랜치 → 규모별 계획 → 트래커 이슈 생성·착수 → 브랜치명에 이슈 키 rename.
- 마무리 요청("머지", "완료 처리")을 하면 → *작업 후* 절차: 테스트 게이트 → 실제 e2e 검증 → 문서 동기화 → 이슈 키 제목의 `--no-ff` 머지 → 이슈 완료 → 워크트리만 제거(브랜치 보존).

신호는 **언어 불문** 인식됩니다. 명시적으로 부르려면: `/work-lifecycle:work-lifecycle`.

워크트리는 저장소 안(기본 `.claude/worktrees/` 하위)에 생성되고 머지 후 제거됩니다 — **브랜치는 항상 보존**되어 롤백 지점으로 남습니다.

위 요약은 골격만입니다. 전체 절차 — 규모 판정표(잔손질/소형/중형/대형과 각각의 계획 형태), 복수 이슈 처리 규칙, 곁가지 백로그 캐처 — 는 [SKILL.md](skills/work-lifecycle/SKILL.md) 에 있습니다.

**프로젝트 어댑터** — 프로젝트의 `CLAUDE.md` 에 아래처럼 정의하면 skill 이 읽어서 적용합니다 (전부 선택 사항, 없으면 기본값):

```markdown
## 작업 워크플로 (work-lifecycle 어댑터)
- 이슈 트래커: Jira, 프로젝트 키 ABC (전환: 진행 중=21, 완료=41)
- 테스트 게이트: `npm test` (머지 전 필수 실행)
- 문서 게이트: 동작 변경 시 docs/ 하위 해당 문서 갱신
- 커밋 컨벤션: 한글, 제목 50자 이내, 제목 끝에 이슈 키
- 도메인 불변: 사용자가 요청하지 않은 가드/필터 추가 금지
```

프로젝트에 자체 절차 정본 문서가 있으면 `CLAUDE.md` 에 그 사실을 명시하세요 — 그 문서가 이 skill 보다 **우선**합니다.

### 사용 예시

```text
사용자:  로그인이 왜 계속 실패하죠?
Claude:  (조사 후 원인 보고만 — 파일 무변경)

사용자:  고쳐주세요.
Claude:  git fetch → 워크트리+브랜치 fix/login-timeout
         → 계획 수립 (소형 → Todo 목록)
         → 트래커에 이슈 ABC-12 생성, "진행 중" 전환
         → 브랜치를 fix/ABC-12-login-timeout 으로 rename
         → 구현 — "fix: … (ABC-12)" 의미 단위 커밋 + 매 커밋 push

사용자:  머지해주세요.
Claude:  테스트 게이트 실행 → 실제 로그인 1회로 e2e 검증
         → 문서 갱신 → merge --no-ff "Merge: ABC-12 로그인 타임아웃 수정"
         → ABC-12 완료 전환 → 워크트리 제거, 브랜치 보존
```

### 보안

이 플러그인에는 **실행 코드가 없습니다** — hook·스크립트·MCP 서버 없이 마크다운 skill 파일 1개뿐이며, 모델에 지침만 추가합니다.

### 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| 설치 직후 skill 이 활성화되지 않음 | 플러그인은 세션 시작 시 로드됩니다 — Claude Code 세션을 재시작하세요 |
| 프로젝트에 이미 자체 워크플로 skill·절차 문서가 있음 | 공존합니다. 프로젝트 자체 정본 문서가 이 skill 보다 우선합니다 (`CLAUDE.md` 에 명시) |
| 이슈 단계가 동작하지 않음 | 이슈 트래커 미연결 — 연결(Atlassian/GitHub 플러그인 + `/mcp` 인증)하거나 Todo 대체로 사용 |
| 일시적으로 끄고 싶음 | `/plugin disable work-lifecycle`, 다시 켜려면 `/plugin enable work-lifecycle` |

### 업데이트

```
/plugin marketplace update wisefool-plugins
/plugin update work-lifecycle
```

### 제거

```
/plugin uninstall work-lifecycle
```

### 변경 이력

[CHANGELOG.md](CHANGELOG.md) 참조.

---

## License

MIT — see [LICENSE](../../LICENSE).
