# Progress Events

После изменения stage/status обновляй:

- `.agent-runs/<change-id>/state.json` — authoritative machine state;
- `.agent-runs/<change-id>/progress.md` — concise human-readable progress.

Не требуется отдельный checkpoint-файл для каждого шага. Resume начинается с `state.json`, Change Context и текущих stage artifacts.
