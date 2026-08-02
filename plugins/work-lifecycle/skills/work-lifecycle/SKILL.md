---
name: work-lifecycle
description: 작업 전·중·후 라이프사이클 절차 준수. 변경 작업 착수 신호("진행합시다", "구현해주세요", "고쳐주세요", "만들어주세요", "작업 시작" / "let's proceed", "go ahead", "implement it", "fix it", "build it", "start working"), 워크트리/브랜치 생성, 이슈 착수, 커밋·push, 머지 요청("머지", "완료 처리", "마무리" / "merge it", "wrap up", "finish it"), 작업 종료 시 자동 활성화. 조사·질문 턴에는 발동하지 않되, 조사 결과에 대해 사용자가 진행을 결정하는 순간 활성화. Activates on work-kickoff and merge/close signals in any language; does not fire on questions or investigation-only turns.
---

# Work Lifecycle Procedure (work-lifecycle)

The common procedure that every task follows — features, bugs, research, refactors, documentation, operations.
**This skill is a generic skeleton.** Project-specific values (issue tracker, verification commands, domain invariants) are **read from that project's CLAUDE.md**, per the "Project adapter" rules below. If the project has its own canonical procedure document (e.g. `docs/ops/work-lifecycle.md`), **that document takes precedence over this skill** — Read it first and follow its procedure.

## Project adapter — what to check in CLAUDE.md

On activation, check the project's CLAUDE.md for the following. Every key is in one of three states — **declared** (follow it), **delegated** (the project explicitly hands the choice to you; decide, then report what you chose), or **undecided** (nothing written).

For most keys, undecided simply means the default in the right-hand column. For **Issue tracker** and **Knowledge locations** — where the user's issues and knowledge are kept — undecided is **not** consent to decide for them: resolve it by asking, once (see "Bootstrapping an undecided key").

| Item | What the project defines | Default when undefined |
|---|---|---|
| Issue tracker | Type (Jira / GitHub Issues / …), project key, how to transition status | Ask once. Once delegated: skip the issue steps; substitute a todo list |
| Test gate | Test commands to run before commit and merge | Run the project's standard test runner once |
| Docs gate | Which documents must stay in sync, and any check command | Check only the user-facing docs of the changed feature |
| Knowledge locations | Where each kind of record is kept — plan/spec documents, incident records, and any other kind the project names. A location may be a repository path (`docs/plans/`), an external space (a wiki, a Confluence space), or the tracker itself. `Plan location: <path>` is also accepted as the plan entry | Ask once. Once delegated: fall back to the issue (description or comment), and to the todo list when there is no tracker. Never create a directory in the project |
| Commit convention | Language, format, prefix rules | Conventional Commits (`fix:`, `feat:`, …) |
| Domain invariants | Principles that must never be violated (e.g. never change specific logic unasked) | None |

### Bootstrapping an undecided key

Where a user's issues and knowledge live is their decision, not something to infer. When one of those two keys is undecided **and the moment to use it arrives**, ask — do not decide quietly.

- **Ask only when needed.** Never run a questionnaire at install time or at the start of a session. Ask about the plan location when the first medium task needs a plan; ask about the incident location when the first incident is closing. A capability that is never used is never asked about.
- **Ask with a proposal, not a blank.** Offer the place you would have chosen, plus "somewhere else" and "decide it for me from now on". Approving or amending a proposal is far easier than answering an open question — and it puts your judgment in front of the user instead of behind them.
- **Always offer delegation.** One answer must be able to switch the question off for good.
- **Write the answer down** in the project's CLAUDE.md adapter. Without that, the next session asks again — and asking twice is where this turns into an annoyance. A user-wide preference can be declared once in the global `~/.claude/CLAUDE.md`; a project's declaration overrides it.

Writing to CLAUDE.md is a privilege with three limits — carry them into the project's domain invariants as well:

- Record **only what the user answered.** Never write down a value you inferred.
- Keep that record as **its own commit**, so it is visible rather than buried in unrelated changes.
- Even when the choice is delegated, **report where the record went.** "Decide it for me" is not "do it behind my back".

### Reaching a declared location

A declared tracker or knowledge location is worth only what this session can actually reach. Confirm it before the work depends on it — the same reason `git fetch origin` comes before the worktree.

- **Confirm at the adapter check**, not at the moment of writing. Confirming means **one real read** against the destination (list the tracker's projects, open the space, resolve the path). The presence of a plugin or tool name is not confirmation.
- **Unreachable is a finding, not a fallback.** Say what could not be reached, propose where the record could go instead, and let the user choose. Falling back silently is worse than never having asked: the record looks written and is not.
- **A record is done only when you can point to it** — an issue URL, a page URL, a committed path. Carry that reference into the issue's result comment. If you cannot produce one, the record did not happen; report that instead of reporting success.

## Terms and criteria

### Task size table

| Size | Criteria | Plan form | Issue |
|---|---|---|---|
| **Trivial** | Typos, comments, a few lines. No design judgment, no behavior change (or an obvious one) | None | **Skipped** |
| **Small** | Scope is clear from the outset. No design decisions | Todo list | Created |
| **Medium** | Multi-step implementation, but the approach is self-evident | Plan document | Created |
| **Large** | The requirements or the design themselves must be explored | Brainstorm → spec → plan | Created |

- When the judgment is ambiguous, treat it as **one size up**.

### Kickoff signal

An explicit statement of the user's intent to proceed — "go ahead", "implement it", "fix it", "build it", and the like.
Questions, investigation, and requests for judgment ("why is this happening?", "is it possible?", "what do you think?") are **not** kickoff signals.

### Substantive work action (the deadline for creating the issue)

Whichever of these happens first: ① an implementation change (product code, configuration, canonical documents) ② a system state change (deployment, DB operations, external service settings) ③ the start of a long-running experiment or research run.
Writing todos or spec/plan documents is **not** a substantive work action (they are planning artifacts).

## 0. Triage (the start of every turn)

| Nature of the request | Handling |
|---|---|
| Question, investigation, root-cause analysis, feasibility judgment | Read-only investigation **without** a worktree or issue, then **report only**. No file changes. Even when a fix is apparent, do not touch it before a kickoff signal |
| Kickoff signal received | Enter **1. Before work** |
| Trivial fix | Skip the issue and the plan. Committing directly on main is allowed when working alone with no risk of collision; use a worktree if concurrent work is in play |
| Side-finding during work (bug, idea) | Do not stop the current task — capture it to the backlog immediately (2.5) |

## 1. Before work (kickoff preparation) — fixed order

1. **Bring main up to date:** `git fetch origin` (required before creating the worktree).
2. **Create the worktree and branch:** work in a worktree based on main. Keep the main working tree clean at all times. If the harness provides a worktree tool (e.g. `EnterWorktree`), use it and take its default location; otherwise create one under `.claude/worktrees/` — `git worktree add .claude/worktrees/<branch-name> -b <branch-name> origin/main`.
3. **Temporary branch name:** `<prefix>/<kebab-case-task-name>` — `feature/` (feature) · `fix/` (defect) · `refactor/` (behavior preserved) · `docs/` (documentation) · `chore/` (chores).
4. **Pre-reads + adapter check:** explicitly Read the reference documents that the project's CLAUDE.md assigns to this kind of work, then read the adapter — resolve an undecided tracker or knowledge location by asking, and confirm that a declared one is reachable (both above).
5. **Plan:** as prescribed by the task size table (todo / plan / brainstorm→spec→plan).
   - **Where the plan lives:** put spec/plan documents in the project's plan location. When that location is a repository path, write them inside the worktree and commit them there; when it is an external space, create them there. Either way they are planning artifacts, not implementation changes, so they may precede the issue. When no location is declared, do **not** invent a directory in that project — carry the plan in the issue description (or the todo list) instead.
6. **Create the issue** (when a tracker is defined):
   - **Timing:** immediately once the plan is confirmed — at the latest, before the first **substantive work action**.
   - For a large task whose planning may outlive the session, create it at kickoff (so an "in progress" record survives even if the session is cut short mid-plan).
   - If the item is already captured in the backlog, do not create a new one — transition it to "in progress" and refresh the description.
   - Title in non-developer terms, outcome-focused; the description carries what and why, plus the branch name and the plan path — or the plan itself, when the project has no plan location.
7. **Rename the branch with the key:** right after creating the issue, `git branch -m <prefix>/<issue-key>-<task-name>`. A branch carries **exactly one key** — the kickoff issue's (no epics, no multiple keys).
   - When planning yields multiple issues: sequential stages = parent + subtasks (parent key) / independent tasks = start only the first and backlog the rest / discovered mid-implementation = narrow the current issue's scope and file a new backlog issue.

## 2. During work (implementation)

1. **Commit discipline:** one commit per meaningful unit (keep it bisectable). Messages follow the project's commit convention, with `(issue-key)` at the end of the subject.
2. **Staging discipline:** blanket `git add .` / `-A` is **forbidden** — stage only your own files, individually, by path. Run `git status` right before committing to confirm that everything staged is yours (in case another session is working concurrently).
3. **Push to the remote:** the first push comes **after** the issue-key rename (`git push -u origin <branch>`). From then on, push **on every commit** (loss protection + external visibility). In a local-only repository with no remote, the push steps are skipped and every other rule applies unchanged.
4. **Notable findings → issue comments:** record design decisions, unexpected behavior, interim measurements, and blockers as they happen.
5. **Capture side-findings immediately:** file unrelated bugs and ideas as backlog issues without breaking your flow, then return. A capture is not subject to the issue-timing rule (it is a record, not a kickoff).
6. **Honor domain invariants:** do not make changes the project's CLAUDE.md forbids (adding guards unasked, and so on). If you judge one to be genuinely necessary, propose it and obtain consent first.

## 3. After work (verify → merge → close)

1. **Test gate:** run the project's defined test and regression commands. On failure, revert the offending commit and rework.
2. **Real end-to-end verification:** declaring "done" on passing mock/unit tests alone is **forbidden**. UI: check the real screen (and ask the user to confirm with their own eyes); API: one real request; CLI: one real run; external integration: one real call.
3. **Docs sync:** update the affected documents and run the project's docs gate.
4. **Merge (after user confirmation):** whether and how to merge is decided with the user.
   ```bash
   git pull origin main && git merge --no-ff <work-branch>
   # after re-confirming the tests
   git push origin main
   ```
   The merge commit title is **issue key + summary**: `Merge: <issue-key> <title>` (so issues remain traceable from main's history). On conflict, resolve it by reverse-merging main into the work branch (never rebase), then retry.
5. **Close the issue:** post a result-summary comment (non-developer terms + commit hashes), then transition the status to done.
6. **Incident record:** if this task was an **incident** — a bug, misconfiguration, or operational mistake that caused *actual* damage: a wrong result reaching users or an external system, data loss or corruption, an outage, or a manual intervention to recover — write one record in the project's incident location. Template: **symptom → cause → fix → prevention → related assets** (issue, commits, canonical documents). Write it for a non-developer reader, and link to canonical documents rather than restating them. Link the record from the issue so it stays traceable. Ordinary feature, refactor and documentation work is **not** an incident, and neither is a near miss — filing those buries the real ones in noise.
7. **Sync memory and records:** if there are lessons worth keeping (findings, feedback, near misses), record them in the project's knowledge locations — its memory and documents. Each record needs a reference you can point to (see "Reaching a declared location").
8. **Clean up the worktree (keep the branch):** `git worktree remove <path>` removes **only the worktree**. The branch is kept both locally and on the remote (history and rollback point).

## Core invariants (summary)

- Never work directly on main (the trivial-fix exception is defined in 0. Triage).
- Never touch another session's worktree or branch.
- The issue-timing rule applies **only to the kickoff issue** — backlog captures happen the moment they are found.
- A branch always carries exactly one key. After merging, the branch is kept and only the worktree is removed.

## Full checklist

```
[Triage]  Question/investigation → report only. Proceed below only on a kickoff signal. Trivial → the issue may be skipped
[Before]  fetch → worktree (based on main) → temporary branch name → pre-reads
          → adapter check (undecided tracker/knowledge location → ask once; declared one → confirm it is reachable) → plan (by size)
          → create issue + in progress (right after the plan is confirmed, before the first substantive action) → rename branch with the key
[During]  meaningful-unit commits (stage only your own files, individually) → first push after the rename, then push on every commit
          → notable findings as issue comments → capture side-findings immediately
[After]   test gate → real e2e verification (never mock-only) → docs sync
          → (user confirmation) merge --no-ff "Merge: <issue-key> <title>" + push
          → close the issue → incident record (only if damage actually occurred) → sync records (each with a reference you can point to)
          → remove only the worktree (keep the branch)
```
