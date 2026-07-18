# wisefool-plugins

A Claude Code plugin marketplace by wisefool. — wisefool 의 Claude Code 플러그인 마켓플레이스.

[English](#english) | [한국어](#한국어)

---

## English

### Install

Inside Claude Code:

```
/plugin marketplace add shincheolhui/wisefool-plugins
/plugin install work-lifecycle@wisefool-plugins
```

### Plugins

#### work-lifecycle

An **opinionated** work-lifecycle skill that enforces a before / during / after procedure in every session:

- **Request triage** — questions and investigations get report-only responses; work starts only on an explicit kickoff signal ("go ahead", "implement it"), with a tiny-fix exception
- **Before** — sync main → worktree + branch → plan by size (todo / plan / spec) → issue-creation timing rule → rename branch with the issue key
- **During** — commit in meaningful units, stage only your own files, push every commit, log decisions as issue comments, capture side-findings to the backlog immediately
- **After** — test gate → real end-to-end verification (no mock-only sign-off) → docs sync → `--no-ff` merge titled with the issue key → close the issue → remove the worktree but keep the branch

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
- Give a kickoff signal ("go ahead", "implement it", "진행합시다") → the skill walks through the *before* steps: fetch → worktree + branch → plan sized to the task → create/start the tracker issue → rename the branch with the issue key.
- Ask to finish ("merge it", "wrap up", "머지") → the *after* steps: test gate → real e2e verification → docs sync → `--no-ff` merge titled with the issue key → close the issue → remove the worktree, keep the branch.

To invoke it explicitly: `/work-lifecycle:work-lifecycle`.

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

### Security

This plugin contains **no executable code** — no hooks, no scripts, no MCP servers. It is a single markdown skill file that only adds instructions to the model.

### Update

```
/plugin marketplace update wisefool-plugins
```

---

## 한국어

### 설치

Claude Code 안에서:

```
/plugin marketplace add shincheolhui/wisefool-plugins
/plugin install work-lifecycle@wisefool-plugins
```

### 플러그인 목록

#### work-lifecycle

작업 전·중·후 라이프사이클 절차를 모든 세션에 강제하는 **의견이 강한(opinionated)** skill.

- **요청 분류** — 질문·조사는 보고만, 착수 신호("진행합시다" 등)에만 작업 진입, 잔손질 예외
- **작업 전** — main 최신화 → 워크트리+브랜치 → 규모별 계획(Todo/plan/spec) → 이슈 생성 시점 규칙 → 브랜치명에 이슈 키 rename
- **작업 중** — 의미 단위 커밋, 내 파일만 개별 스테이징, 매 커밋 push, 특이사항 이슈 댓글, 곁가지 즉시 백로그 캐처
- **작업 후** — 테스트 게이트 → 실제 실행 e2e 검증(mock 금지) → 문서 동기화 → `--no-ff` 머지(이슈 키 제목) → 이슈 완료 → 워크트리만 제거(브랜치 보존)

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

명시적으로 부르려면: `/work-lifecycle:work-lifecycle`.

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

### 보안

이 플러그인에는 **실행 코드가 없습니다** — hook·스크립트·MCP 서버 없이 마크다운 skill 파일 1개뿐이며, 모델에 지침만 추가합니다.

### 업데이트

```
/plugin marketplace update wisefool-plugins
```

---

## License

MIT — see [LICENSE](LICENSE).
