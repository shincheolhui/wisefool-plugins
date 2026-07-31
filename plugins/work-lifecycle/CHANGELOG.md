# Changelog

## work-lifecycle 1.2.0 (2026-07-31)

- 어댑터 키 추가: **계획 문서 위치** / New adapter key: **Plan location** — spec·plan 문서를 둘 디렉토리를 프로젝트 CLAUDE.md 에서 선언한다. Declare in the project's CLAUDE.md where spec/plan documents are kept.
- 계획 문서 소재 규정 / Plan documents now have a defined home — 워크트리 안에서 작성·커밋하며, 계획 산출물이므로 이슈 생성 전에 써도 된다. Written and committed inside the worktree; being a planning artifact, it may precede the issue.
- 위치 미선언 시 동작 / When no location is declared — 프로젝트에 디렉토리를 새로 만들지 않고 계획을 이슈 설명(또는 Todo)에 담는다. The skill will not create a directory in your project; the plan is carried in the issue description or the todo list instead.
- 배경: 1.6 이 이슈 설명에 "plan 경로"를 요구하는데 그 경로의 소재가 정본에 없었다 / Background: step 1.6 required a "plan path" in the issue description while the canon never said where that path should be.

## work-lifecycle 1.1.0 (2026-07-30)

- SKILL.md 본문을 영어로 단일화 / SKILL.md body is now English-only — 절차·구조·의미는 불변, 한국어 착수 신호 트리거(frontmatter `description`)는 그대로 유지. Procedure, structure and meaning unchanged; the Korean kickoff-signal triggers are kept.
- 워크트리 위치 규정 추가 / Worktree location specified — 하네스 워크트리 도구가 있으면 그 기본 위치, 없으면 `.claude/worktrees/` 하위. Use the harness worktree tool's default location, otherwise create it under `.claude/worktrees/`.
- 원격 없는 저장소 규정 추가 / Local-only repositories specified — `origin` 이 없으면 push 단계만 생략하고 나머지 규칙은 불변. With no remote, only the push steps are skipped.
- 위 두 규칙은 README 가 이미 약속하고 있었으나 정본에 없던 것을 명문화한 것 / Both rules were already promised by the README but absent from the canon.

## work-lifecycle 1.0.0 (2026-07-18)

- 최초 공개 / Initial release
- 작업 전·중·후 라이프사이클 skill (요청 분류 → 워크트리·브랜치 → 규모별 계획 → 이슈 시점 규칙 → 커밋·push 규율 → 검증·머지·종료)
- 프로젝트 어댑터: 이슈 트래커·테스트 게이트·커밋 컨벤션·도메인 불변을 각 프로젝트 CLAUDE.md 에서 읽음, 자체 정본 문서 우선
