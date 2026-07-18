# shincheolhui-plugins

shincheolhui 의 Claude Code 플러그인 마켓플레이스.

## 설치

Claude Code 안에서:

```
/plugin marketplace add shincheolhui/shincheolhui-plugins
/plugin install work-lifecycle@shincheolhui-plugins
```

## 플러그인 목록

### work-lifecycle

작업 전·중·후 라이프사이클 절차를 모든 세션에 강제하는 skill.

- **요청 분류** — 질문·조사는 보고만, 착수 신호("진행합시다" 등)에만 작업 진입, 잔손질 예외
- **작업 전** — main 최신화 → 워크트리+브랜치 → 규모별 계획(Todo/plan/spec) → 이슈 생성 시점 규칙 → 브랜치명에 이슈 키 rename
- **작업 중** — 의미 단위 커밋, 내 파일만 개별 스테이징, 매 커밋 push, 특이사항 이슈 댓글, 곁가지 즉시 백로그 캐처
- **작업 후** — 테스트 게이트 → 실제 실행 e2e 검증(mock 금지) → 문서 동기화 → `--no-ff` 머지(이슈 키 제목) → 이슈 완료 → 워크트리만 제거(브랜치 보존)

프로젝트별 값(이슈 트래커·테스트 명령·커밋 컨벤션·도메인 불변)은 각 프로젝트의 `CLAUDE.md` 에 정의하면 skill 이 읽어서 적용합니다("프로젝트 어댑터" — SKILL.md 참조). 정의가 없으면 안전한 기본값으로 동작합니다.

프로젝트에 자체 절차 정본 문서가 있으면 그 문서가 이 skill 보다 우선합니다.

## 업데이트

```
/plugin marketplace update shincheolhui-plugins
```

## 라이선스

MIT
