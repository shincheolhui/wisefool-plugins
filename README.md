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

Project-specific values (issue tracker, test commands, commit convention, domain invariants) are read from each project's `CLAUDE.md` via the **project adapter** section in SKILL.md; safe defaults apply when undefined. If a project has its own canonical procedure document, that document takes precedence over this skill.

### Requirements

- [Claude Code](https://claude.com/claude-code)
- A git repository (the workflow is built around branches and worktrees)
- Optional: an issue tracker reachable from Claude Code (e.g. Jira/GitHub MCP) — without one, the issue steps fall back to todo lists

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

프로젝트별 값(이슈 트래커·테스트 명령·커밋 컨벤션·도메인 불변)은 각 프로젝트의 `CLAUDE.md` 에 정의하면 skill 이 읽어서 적용합니다("프로젝트 어댑터" — SKILL.md 참조). 정의가 없으면 안전한 기본값으로 동작합니다. 프로젝트에 자체 절차 정본 문서가 있으면 그 문서가 이 skill 보다 우선합니다.

### 요구사항

- [Claude Code](https://claude.com/claude-code)
- git 저장소 (워크플로가 브랜치·워크트리를 전제)
- 선택: Claude Code 에서 접근 가능한 이슈 트래커(Jira/GitHub MCP 등) — 없으면 이슈 단계는 Todo 목록으로 대체됩니다

### 보안

이 플러그인에는 **실행 코드가 없습니다** — hook·스크립트·MCP 서버 없이 마크다운 skill 파일 1개뿐이며, 모델에 지침만 추가합니다.

### 업데이트

```
/plugin marketplace update wisefool-plugins
```

---

## License

MIT — see [LICENSE](LICENSE).
