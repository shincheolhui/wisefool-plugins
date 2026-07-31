# CLAUDE.md — wisefool-plugins

Claude Code 플러그인 마켓플레이스 저장소. 실행 코드 없이 마크다운 정의만 담는다.

## 저장소 구조

| 경로 | 역할 |
|---|---|
| `.claude-plugin/marketplace.json` | 마켓플레이스 카탈로그 (플러그인 목록·설명) |
| `plugins/<name>/.claude-plugin/plugin.json` | 플러그인 메타 (버전·author·keywords) |
| `plugins/<name>/skills/<name>/SKILL.md` | **정본** — 모델이 실제로 따르는 것 그 자체 |
| `plugins/<name>/README.md` | 사용자 문서 (영어·한국어 이중언어) |
| `plugins/<name>/CHANGELOG.md` | 변경 이력 |

## 작업 워크플로 (work-lifecycle 어댑터)

작업 전·중·후 절차는 **이 저장소가 배포하는 work-lifecycle skill 을 그대로 따른다** (자기 도구를 자기가 쓴다).
절차 정본: [plugins/work-lifecycle/skills/work-lifecycle/SKILL.md](plugins/work-lifecycle/skills/work-lifecycle/SKILL.md)

- **이슈 트래커:** GitHub Issues — 이 저장소 (`gh` CLI 또는 GitHub MCP 로 조작).
  - 이슈 키 표기는 `#N`, 브랜치는 `<접두사>/<N>-<슬러그>` (예: `docs/1-plugin-independence`)
  - "진행 중" = 이슈 open + 담당자 배정 / "완료" = 결과 요약 댓글을 남긴 뒤 close
- **테스트 게이트:** 머지 전 필수. 매니페스트 유효성 검증 —
  ```bash
  claude plugin validate . --strict                   # 마켓플레이스 매니페스트
  claude plugin validate plugins/<name> --strict      # 플러그인 매니페스트
  ```
- **문서 게이트:** SKILL.md 를 고쳤으면 ① 같은 플러그인 README 와 전수 대조해 불일치 0건 확인
  ② CHANGELOG 항목 추가 ③ 사용자 가시 변경이면 `plugin.json` version 올림 (SemVer).
- **지식 기록 위치:** 계획 → **선언하지 않음**(이슈에 담는다 — #3 에서 선언했다가 되돌린 판단,
  근거는 `5f6d0c9`). 사고 기록 → 아직 미정 (사고가 처음 났을 때 묻고 정한다).
- **커밋 컨벤션:** 한글, Conventional 접두사(`docs:` `fix:` `feat:` `refactor:` `chore:`),
  제목 50자 내외 + 끝에 `(#N)`. 본문은 불릿으로 무엇을·왜.
- **도메인 불변:**
  - **이 CLAUDE.md 에 어댑터 값을 쓸 때는 사용자가 실제로 답한 것만 적는다.** 추론한 값을
    적지 않고, 그 기록은 별도 커밋으로 분리하며, 위임된 항목이라도 결과 경로는 보고한다.
  - **SKILL.md 가 유일한 정본이다.** README 는 절차를 재서술하지 않고 SKILL.md 를 가리킨다 —
    같은 규칙을 두 곳에 쓰면 반드시 어긋난다 (실제로 #1 에서 2건 발생했다).
  - **플러그인에 실행 코드(hook·script·MCP 서버)를 추가하지 않는다.** README 보안 섹션이
    "설치해도 사용자 컴퓨터에서 아무것도 실행되지 않는다"를 보증하고 있다. 추가가 필요하면
    먼저 제안하고, 승인되면 그 보증 문구부터 함께 고친다.
  - SKILL.md frontmatter `description` 의 **한국어 트리거 문구를 제거하지 않는다** —
    본문은 영어 정본이지만, 한국어 사용자의 skill 발동이 이 문구에 걸려 있다.

## 이 저장소 이력에 대한 주의

커밋 `6c96e15`~`e2eff4d` 에 붙은 `LETF-44` 키는 오기가 아니다. 이 저장소는 Leveraged ETF trading
프로젝트의 이슈 LETF-44("작업 절차를 다른 프로젝트·다른 사람도 쓸 수 있게 플러그인으로 배포")의
**산출물로 태어났고**, 그 이슈가 완료된 뒤 독립 프로젝트로 졸업했다. 그래서 초기 커밋이 남의
프로젝트 키를 달고 있는 것은 당시 절차상 정확했다. `#1` 이후의 작업은 위 어댑터대로 이 저장소의
GitHub Issues 를 쓴다.
