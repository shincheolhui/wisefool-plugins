# Changelog

## work-lifecycle 1.6.0 (2026-08-03)

1.4.0 은 "안 정해졌으면 대신 정하지 말고 묻는다"를 보장했다. 이번에는 그 다음 사고를 막는다 — **정한 곳에 실제로 닿지 못하는 경우.** / 1.4.0 guaranteed the skill would ask rather than choose. This release closes what came next: a location that was chosen but cannot be reached.

- **도달성 확인 신설** / New rule — `Reaching a declared location`. 선언된 트래커·지식 위치는 **쓰는 순간이 아니라 어댑터 확인 시점에**, 목적지에 대한 **실제 읽기 1회**로 확인한다. 플러그인·도구 이름이 있다는 것은 확인이 아니다. Confirmed with one real read at the adapter check — the presence of a plugin name is not confirmation.
- **도달 실패는 폴백이 아니라 발견 사항** / Unreachable is a finding, not a fallback — 무엇에 못 닿았는지 말하고 대안을 제안한 뒤 사용자가 고른다. 조용한 폴백은 원래 문제보다 나쁘다: 기록이 쓰인 것처럼 보이지만 없다. Falling back silently is worse than never having asked.
- **가리킬 수 있어야 기록이 끝난 것** / A record is done only when you can point to it — 이슈 URL·페이지 URL·커밋된 경로를 이슈 결과 댓글에 싣는다. 참조를 못 내놓으면 성공 대신 "기록이 일어나지 않았다"고 보고한다. If no reference can be produced, that is reported instead of success.
- 배경 / Background — README 는 전제조건과 어댑터 1·4번, **세 곳에서** "실제로 접근 가능해야 한다"고 약속해 왔는데 정본에는 도달성이라는 개념 자체가 없었다. The README had promised this in three places while the canon never mentioned reachability.
- 부수 / Also — `1.4` 가 `Pre-reads + adapter check` 로 확장됐다. 체크리스트에만 있고 번호 절차에는 없던 어댑터 확인 단계가 정본에 명시된다. The adapter check existed only in the checklist, never in the numbered steps.

## work-lifecycle 1.5.0 (2026-07-31)

문서만 바뀐 릴리스다. 절차 자체는 1.4.0 과 동일하되, 제품 설명이 실체를 따라오지 못한 부분을 맞췄다. / A documentation release: the procedure is unchanged from 1.4.0, but the product description now matches what the skill actually does.

- **소개문 4곳 개정** / Product description refreshed — `marketplace.json`·`plugin.json`·저장소 README·플러그인 README 가 1.0.0 문구("triage, isolated worktree, tracked issue, verified merge")에 멈춰 있어, 1.2.0~1.4.0 에서 더해진 지식 축이 빠져 있었다. All four were still describing 1.0.0.
- **"왜 필요한가"에 지식 축 추가** / Sixth problem/guarantee pair — 작업이 남긴 것이 대화에만 남아 다음 세션이 아무것도 모른 채 시작하는 문제와, 기록이 사용자가 정한 자리에 남는다는 보장. Records land where you decided they belong.
- **전제조건에 기록 저장소 도달 가능성** / New prerequisite — 선언했는데 도달하지 못하는 위치는 규칙을 조용히 실패시킨다. A declared but unreachable location makes the rule fail silently.
- 체크리스트 `[Before]` 에 어댑터 미정 해소 단계 / Checklist now shows the adapter check.

## work-lifecycle 1.4.0 (2026-07-31)

- **어댑터 3상태 모델** / Adapter keys now have three states — 선언 / 위임 / 미정. 지금까지는 "미선언"을 "AI 가 알아서 정해도 좋다"로 해석했는데, *아직 아무도 안 정했다* 와 *알아서 해줘* 는 다른 상태다. Until now "not declared" was read as consent to decide; *undecided* and *delegated* are different states.
- **미정 키 부트스트랩** / Bootstrapping an undecided key — 이슈 트래커·지식 기록 위치가 미정이면 조용히 정하지 않고 **그 답이 처음 필요해지는 순간 한 번 묻고, 답을 CLAUDE.md 에 기록**해 다시 묻지 않는다. For those two keys the skill asks once, at the moment the answer is first needed, and records it.
- 질문 제약 / How it asks — 설치 직후 설문 금지, 제안과 함께 묻기, 위임 선택지 상시 제공, 답은 반드시 기록. No questionnaire at install time; ask with a proposal; always offer delegation; always write the answer down.
- 전역 선언 / User-wide preference — `~/.claude/CLAUDE.md` 에 한 번 선언하면 모든 프로젝트에 적용되고 프로젝트 선언이 덮어쓴다. Declare once globally; a project declaration overrides it.
- **쓰기 한계 3가지** / Three limits on writing to CLAUDE.md — 물어서 받은 답만 기록 · 별도 커밋으로 분리 · 위임해도 결과 경로는 보고. Only what the user answered; its own commit; report where records went even when delegated.

## work-lifecycle 1.3.0 (2026-07-31)

- 어댑터 키 `Plan location` → **`Knowledge locations`** 로 확장 / adapter key widened — 기록 종류별로 소재를 선언한다. 소재는 저장소 경로·외부 공간(위키·Confluence)·트래커 자신 중 무엇이든 될 수 있다. Declare where each kind of record is kept; a location may be a repository path, an external space, or the tracker itself.
- **호환** / Compatibility — 1.2.0 의 `Plan location: <경로>` 표기는 계획 항목 선언으로 계속 인정된다. `Plan location: <path>` is still accepted as the plan entry.
- 계획 문서 소재 조건화 / Plan documents no longer assume a repository path — 소재가 저장소 경로일 때만 워크트리 안에서 작성·커밋하고, 외부 공간이면 거기에 생성한다. Written and committed in the worktree only when the location is a repository path.
- **사고 기록 조항 신설 (3.6)** / New step — incident record — 버그·설정 오류·운영 실수로 **실제 피해가 발생한** 작업은 `증상 → 원인 → 해결 → 재발 방지 → 관련 자산` 기록을 남기고 이슈에서 링크한다. 일반 작업과 near miss 는 대상이 아니다(노이즈가 진짜 사고를 묻는다). Only for tasks that caused actual damage; ordinary work and near misses are excluded.
- 기존 3.6 교훈 → 3.7, 3.7 워크트리 정리 → 3.8 재번호 / Subsequent steps renumbered.

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
