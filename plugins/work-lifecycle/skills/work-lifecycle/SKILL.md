---
name: work-lifecycle
description: 작업 전·중·후 라이프사이클 절차 준수. 변경 작업 착수 신호("진행합시다", "구현해주세요", "고쳐주세요", "만들어주세요", "작업 시작" / "let's proceed", "go ahead", "implement it", "fix it", "build it", "start working"), 워크트리/브랜치 생성, 이슈 착수, 커밋·push, 머지 요청("머지", "완료 처리", "마무리" / "merge it", "wrap up", "finish it"), 작업 종료 시 자동 활성화. 조사·질문 턴에는 발동하지 않되, 조사 결과에 대해 사용자가 진행을 결정하는 순간 활성화. Activates on work-kickoff and merge/close signals in any language; does not fire on questions or investigation-only turns.
---

# Work Lifecycle Procedure (work-lifecycle)

The common procedure that every task follows — features, bugs, research, refactors, documentation, operations.
**This skill is a generic skeleton.** Project-specific values (issue tracker, verification commands, domain invariants) are **read from that project's CLAUDE.md**, per the "Project adapter" rules below. If the project has its own canonical procedure document (e.g. `docs/ops/work-lifecycle.md`), **that document takes precedence over this skill** — Read it first and follow its procedure.

## Project adapter — what to check in CLAUDE.md

On activation, check the project's CLAUDE.md for the following. When a key is undefined, operate on the default in the right-hand column.

| Item | What the project defines | Default when undefined |
|---|---|---|
| Issue tracker | Type (Jira / GitHub Issues / …), project key, how to transition status | Skip the issue steps; substitute a todo list |
| Test gate | Test commands to run before commit and merge | Run the project's standard test runner once |
| Docs gate | Which documents must stay in sync, and any check command | Check only the user-facing docs of the changed feature |
| Commit convention | Language, format, prefix rules | Conventional Commits (`fix:`, `feat:`, …) |
| Domain invariants | Principles that must never be violated (e.g. never change specific logic unasked) | None |

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
2. **Create the worktree and branch:** work in a worktree based on main. Keep the main working tree clean at all times.
3. **Temporary branch name:** `<prefix>/<kebab-case-task-name>` — `feature/` (feature) · `fix/` (defect) · `refactor/` (behavior preserved) · `docs/` (documentation) · `chore/` (chores).
4. **Pre-reads + skill activation:** explicitly Read the reference documents that the project's CLAUDE.md assigns to this kind of work.
5. **Plan:** as prescribed by the task size table (todo / plan / brainstorm→spec→plan).
6. **Create the issue** (when a tracker is defined):
   - **Timing:** immediately once the plan is confirmed — at the latest, before the first **substantive work action**.
   - For a large task whose planning may outlive the session, create it at kickoff (so an "in progress" record survives even if the session is cut short mid-plan).
   - If the item is already captured in the backlog, do not create a new one — transition it to "in progress" and refresh the description.
   - Title in non-developer terms, outcome-focused; the description carries what and why, plus the branch name and the plan path.
7. **Rename the branch with the key:** right after creating the issue, `git branch -m <prefix>/<issue-key>-<task-name>`. A branch carries **exactly one key** — the kickoff issue's (no epics, no multiple keys).
   - When planning yields multiple issues: sequential stages = parent + subtasks (parent key) / independent tasks = start only the first and backlog the rest / discovered mid-implementation = narrow the current issue's scope and file a new backlog issue.

## 2. During work (implementation)

1. **Commit discipline:** one commit per meaningful unit (keep it bisectable). Messages follow the project's commit convention, with `(issue-key)` at the end of the subject.
2. **Staging discipline:** blanket `git add .` / `-A` is **forbidden** — stage only your own files, individually, by path. Run `git status` right before committing to confirm that everything staged is yours (in case another session is working concurrently).
3. **Push to the remote:** the first push comes **after** the issue-key rename (`git push -u origin <branch>`). From then on, push **on every commit** (loss protection + external visibility).
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
6. **Sync memory and records:** if there are lessons worth keeping (incidents, findings, feedback), record them in the project's memory and documents.
7. **Clean up the worktree (keep the branch):** `git worktree remove <path>` removes **only the worktree**. The branch is kept both locally and on the remote (history and rollback point).

## Core invariants (summary)

- Never work directly on main (the trivial-fix exception is defined in 0. Triage).
- Never touch another session's worktree or branch.
- The issue-timing rule applies **only to the kickoff issue** — backlog captures happen the moment they are found.
- A branch always carries exactly one key. After merging, the branch is kept and only the worktree is removed.

## Full checklist

```
[Triage]  Question/investigation → report only. Proceed below only on a kickoff signal. Trivial → the issue may be skipped
[Before]  fetch → worktree (based on main) → temporary branch name → pre-reads → plan (by size)
          → create issue + in progress (right after the plan is confirmed, before the first substantive action) → rename branch with the key
[During]  meaningful-unit commits (stage only your own files, individually) → first push after the rename, then push on every commit
          → notable findings as issue comments → capture side-findings immediately
[After]   test gate → real e2e verification (never mock-only) → docs sync
          → (user confirmation) merge --no-ff "Merge: <issue-key> <title>" + push
          → close the issue → sync records → remove only the worktree (keep the branch)
```
