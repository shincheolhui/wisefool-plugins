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
| [work-lifecycle](plugins/work-lifecycle/) | Makes Claude Code work like a disciplined teammate — triage, isolated worktree, tracked issue, verified merge — and keeps what each task taught you in the place you chose, asking once instead of deciding for you. — Claude Code 를 규율 있는 팀 동료처럼 — 질문엔 조사만, 착수엔 격리 워크트리→이슈 추적→검증된 머지. 작업이 남긴 지식은 사용자가 정한 자리에 남기고, 정해지지 않았으면 대신 정하지 않고 한 번 묻습니다. | `/plugin install work-lifecycle@wisefool-plugins` |

Each plugin's documentation (requirements, usage, troubleshooting, changelog) lives in its own directory under [`plugins/`](plugins/). — 각 플러그인의 문서(요구사항·사용법·문제 해결·변경 이력)는 [`plugins/`](plugins/) 하위 각자의 디렉토리에 있습니다.

## Update the marketplace — 마켓플레이스 업데이트

```
/plugin marketplace update wisefool-plugins
```

## License

MIT — see [LICENSE](LICENSE).
