# QA FLOW FOR SDD v.1.0 — Release Notes

## Current v1.0 baseline

Пакет зафиксирован с учётом пилота на реальной проектной задаче.

### Checklist

- `checklist.md` — человекочитаемый список «что проверить» для любого члена команды;
- technical traceability хранится в `checklist.json`;
- checklist не является independent coverage.

### Project Test Case Contract

- `test-case-profile.yaml` задаёт обязательные/необязательные поля, steps, test data и expected-result model;
- `test-case-writing-rules.md` задаёт atomicity и стиль;
- writer/maintainer не угадывают project format;
- при ненастроенном profile используется `TEST_CASE_PROFILE_NOT_CONFIGURED`.

### New vs existing test cases

Manual test changes разделены на два независимых результата:

1. `new-test-cases.md/json` — только новые полные test cases для CREATE;
2. `test-case-updates.md/json` — только existing cases, которым нужен UPDATE/PROPOSE_OBSOLETE.

Update plan содержит exact identity и field/step-level patch operations с `before → after`, reason и source refs. Это позволяет видеть конкретные изменения без полного переписывания existing case.

### Atomicity

- один case = одна primary objective + один business scenario + один основной outcome;
- mega-cases должны быть split до Human Gate;
- split существующего case сохраняет его identity только за одной responsibility, остальные scenarios становятся новыми cases.

### Pre-export validation

Coverage проверяется по effective test set:

`current existing cases + approved update patches + approved new test cases`.

Проверяются integrity, atomicity, traceability, mandatory risk coverage, update baseline guards и TMS field compatibility.

### TMS handoff

- `tms-export` формирует `export-plan.json`, но не выполняет push;
- внешний IDE plugin является executor;
- CREATE берётся только из `new-test-cases.json`;
- UPDATE — только из `test-case-updates.json`;
- mismatch `before` value должен давать conflict, а не silent overwrite;
- `PROPOSE_OBSOLETE` не означает automatic delete;
- `tms-mapping.json` обновляется только подтверждёнными успешными результатами.

### Runtime

Runtime структура: `state.json`, `progress.md`, `context/`, `clarifications/`, `artifacts/`. Human Gate decisions сохраняются в state.

### Existing test case discovery from test model + Zephyr

После пилота усилено определение existing cases для UPDATE:

- `test-model-impact-analysis` разделён на Candidate Discovery и Candidate Inspection;
- primary discovery выполняется по существующему полю `Тип проверки` в текущем `test-model.md`;
- из matrix rows берутся exact Zephyr/TMS keys;
- создаются `test-case-candidates.md/json` с причинами выбора;
- actual current cases читаются через Jira/Zephyr read integration;
- только после чтения body разрешены `KEEP / UPDATE / OBSOLETE / REVIEW`;
- при downstream impact выполняется secondary related search по текущим matrix rows;
- `test-case-maintainer` больше не делает самостоятельный discovery, а только формирует точный patch по approved identity + retrieved baseline;
- `test-model.md` в package не изменён и новый config для поля `Тип проверки` не добавлялся.

- actual retrieved existing cases сохраняются в `existing-test-cases-snapshot.json`, чтобы downstream update patches оставались воспроизводимыми между сессиями.
