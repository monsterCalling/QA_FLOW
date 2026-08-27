# .agent-runs

Рабочее состояние QA flow по change. Папка не является финальным QA deliverable и обычно не коммитится.

```text
.agent-runs/<change-id>/
├── state.json          # authoritative state + Human Gate decisions
├── progress.md         # human-readable status
├── context/            # selective context manifests
├── clarifications/     # USER_CONFIRMED Change Context
└── artifacts/          # stage drafts до approval
```

Approved artifacts публикуются отдельно в `openspec/changes/<change-id>/qa/`.
