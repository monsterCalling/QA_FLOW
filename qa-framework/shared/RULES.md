# Shared QA Agent Rules

Эти правила обязательны для `qa-flow` и всех stage/helper skills.

## Источники и приоритет

Используй доказательства в порядке:

1. Current OpenSpec Change (`proposal.md`, `preview.md`, `spec.md`/`specs/**`).
2. Подтверждённые человеком clarification answers текущего change.
3. Human-approved upstream QA artifacts этого же change.
4. Project Context.
5. QA Context.
6. Existing test model / manual / auto artifacts.
7. Repository/runtime evidence.
8. Общие знания модели — только для гипотез, техник и вопросов, но не как факт проекта.

Если источники конфликтуют — не выбирай молча. Зафиксируй конфликт и запроси clarification.

## Anti-hallucination

Нельзя придумывать business rules, statuses, API/Kafka/DB contracts, retry/timeout/idempotency semantics, permission rules, Zephyr keys, test data, code paths или возможности окружения.

Всегда различай `FACT`, `INFERENCE`, `ASSUMPTION`, `UNKNOWN`.

`ASSUMPTION` не может стать final expected behavior, defect, test expectation или verdict без human confirmation.

## Human approval

Ни один skill, включая `qa-flow`, не имеет права сам поставить Human Gate в `APPROVED`. Approval существует только если пользователь явно подтвердил его в диалоге или он уже записан как подтверждённый external decision.

## Scope

Работай по текущему change и доказанному regression impact. Не выполняй полный recursive scan repository/Context Pack без явной причины и Human approval.

## Output

Stage обычно создаёт `.md` + `.json`. В артефакты не включай скрытые chain-of-thought; сохраняй evidence, краткое rationale, decisions, findings и traceability.

## Manual test / TMS invariants

- Manual test format определяется project `test-case-profile.yaml`, а не общим шаблоном модели.
- Approved `new-test-cases.json` и `test-case-updates.json` нельзя семантически переписывать в export stage.
- Checklist не считается independent test coverage.
- TMS identity и test data нельзя придумывать.
