# Context Contract

## Слои контекста

Каждый stage использует минимально достаточное пересечение:

1. **Current Change** — обязательные analyst artifacts и релевантные optional artifacts.
2. **Project Context** — только релевантные sections/anchors/symbols.
3. **QA Context** — только rules/techniques, применимые к текущему stage/change.
4. **Current Run Context** — confirmed clarifications + approved upstream artifacts + current state/artifacts.

## Запрещённый режим

Не передавай stage весь `qa-context/`, весь `test-model.md` или весь repository «на всякий случай».

## Selection pipeline

`context-discovery` → `change-signals.json` → `context-catalog.yaml` + skill needs → `context-manifest.json` → selective read → stage.

Каждый selected source должен иметь `selector`, `reason` и `priority`.

Допустимые selectors: `FULL_FILE` (только небольшой обязательный файл), `SECTION`, `ANCHOR`, `SEARCH_THEN_RANGE`, `SYMBOL`, `REQUIREMENT_IDS`, `TEST_MODEL_ANCHORS`.

## Expansion

Если evidence недостаточно, stage может запросить точечное расширение контекста. Расширение записывается в manifest с причиной. Если authoritative context не найден и это влияет на решение — `WAITING_INPUT`.
