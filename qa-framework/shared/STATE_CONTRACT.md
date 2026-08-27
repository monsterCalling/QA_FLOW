# State Contract

`state.json` — внешняя память QA-flow. LLM session может исчезнуть; state должен позволять восстановить процесс.

## Canonical location

`.agent-runs/<change-id>/state.json`

## Stage statuses

`PENDING`, `IN_PROGRESS`, `WAITING_INPUT`, `WAITING_HUMAN`, `WAITING_EXTERNAL`, `COMPLETED`, `COMPLETED_WITH_FINDINGS`, `REJECTED`, `BLOCKED`, `INVALIDATED`, `SKIPPED`.

`COMPLETED_WITH_FINDINGS` означает: stage успешно завершён, non-blocking LOW/TRIVIAL findings/risks сохранены. Для `spec-review` этот статус ставится только **после explicit confirmation Human Gate**; для stages, где policy разрешает auto-continue, он может ставиться автоматически.

## Overall flow statuses

`IN_PROGRESS`, `WAITING_INPUT`, `WAITING_HUMAN`, `WAITING_EXTERNAL`, `COMPLETED`, `BLOCKED`.

## Mandatory behavior

- перед stage: поставить `IN_PROGRESS`;
- после artifact: записать artifact paths и priority summary, если stage его формирует;
- при blocking clarification: `WAITING_INPUT`;
- во время interactive clarification round (`spec-review`, `risk-analysis`, `test-model-impact-analysis`, `test-design`, `spec-vs-code`, `test-failure-analysis`) допускается временно поставить `WAITING_INPUT`; после ответа сохранить Change Context и вернуть текущий stage в `IN_PROGRESS`;
- при активированном Human Gate: `WAITING_HUMAN` и STOP;
- `APPROVED`/accepted risk фиксировать только после explicit human action;
- если только LOW/TRIVIAL items и policy разрешает auto-continue для данного stage: stage → `COMPLETED_WITH_FINDINGS`;
- исключение: `spec-review` всегда → `WAITING_HUMAN` с `gate.type=CONFIRMATION_GATE` до explicit `APPROVE_AND_CONTINUE`;
- `BLOCKED` автоматически использовать только для hard-block policy (например CRITICAL finding) либо explicit human BLOCK;
- при rework upstream invalidировать dependent stages;
- при resume сначала читать state, а не полагаться на историю чата.

## Проверка состояния

`STATE_CONTRACT.md` является семантическим source of truth для состояния, а `qa-framework/templates/state.template.json` — структурным образцом. Перед записью state controller обязан сверяться с обоими файлами и не вводить произвольные статусы или поля с иной семантикой.

## Stage-specific result mapping

`pre-export-validation` может вернуть result `PASSED`; controller фиксирует stage state как `COMPLETED`. Result `BLOCKED` → state `BLOCKED`. `tms-export` после подготовки lossless plan имеет stage state `COMPLETED` и data/handoff marker `READY_FOR_EXTERNAL_EXPORT`.
