# wisefool-plugins

A Claude Code plugin marketplace by wisefool. — wisefool 의 Claude Code 플러그인 마켓플레이스.

## Add this marketplace — 마켓플레이스 추가

Inside Claude Code — Claude Code 안에서:

```
/plugin marketplace add shincheolhui/wisefool-plugins
```

## Plugins — 플러그인 목록

| Plugin | Description | Install |
|---|---|---|
| [work-lifecycle](plugins/work-lifecycle/) | An opinionated before / during / after work-lifecycle skill: request triage, worktree/branch discipline, issue tracking, commit/push rules, verify-merge-close procedure. — 작업 전·중·후 라이프사이클 절차 skill: 요청 분류, 워크트리·브랜치 규율, 이슈 추적, 커밋·push 규율, 검증·머지·종료 | `/plugin install work-lifecycle@wisefool-plugins` |

Each plugin's documentation (requirements, usage, troubleshooting, changelog) lives in its own directory under [`plugins/`](plugins/). — 각 플러그인의 문서(요구사항·사용법·문제 해결·변경 이력)는 [`plugins/`](plugins/) 하위 각자의 디렉토리에 있습니다.

## Update the marketplace — 마켓플레이스 업데이트

```
/plugin marketplace update wisefool-plugins
```

## License

MIT — see [LICENSE](LICENSE).
