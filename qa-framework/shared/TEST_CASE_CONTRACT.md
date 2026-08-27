# Test Case Contract

## Two-artifact model

QA flow разделяет изменения manual tests на два независимых канонических артефакта:

1. `new-test-cases.json` / `new-test-cases.md` — **только новые** test cases (`CREATE`);
2. `test-case-updates.json` / `test-case-updates.md` — **только изменения существующих** test cases (`UPDATE` / `PROPOSE_OBSOLETE`) с точными patch operations.

Нельзя смешивать новый case и update существующего в одном списке.

## Project profile

Writer/maintainer обязаны читать:

- `qa-context/qa/test-cases/test-case-profile.yaml`;
- `qa-context/qa/test-cases/test-case-writing-rules.md`;
- `qa-context/qa/test-cases/test-case-examples.md` — approved style examples;
- `qa-context/qa/test-cases/tms-field-mapping.yaml` — export compatibility.

Если `project_configured != true`, stage → `WAITING_INPUT: TEST_CASE_PROFILE_NOT_CONFIGURED`.

## New cases

`new-test-cases.json` — canonical source для CREATE. `new-test-cases.md` — human projection.

Новый case имеет stable local ID и `tms_key = null` до фактического ответа TMS.

## Existing-case updates

`test-case-updates.json` — canonical patch source для UPDATE существующих cases. `test-case-updates.md` показывает человеку, какой TC и что именно менять.

Каждый UPDATE обязан иметь:

- exact identity (`tms_key` и/или stable local ID);
- current title;
- reason;
- `fields_affected`;
- individual `patch_operations`;
- `before` и `after` для каждого изменения;
- source refs;
- affected test conditions.

`before` служит optimistic guard. Несовпадение current value при export = conflict, а не silent overwrite.

## Atomicity

Один test case = одна primary objective + один business scenario + один основной outcome.

API/DB/Kafka/log assertions не требуют отдельного case, если подтверждают тот же outcome.

Split existing case:

- existing identity остаётся у одной responsibility;
- новые responsibilities создаются через `new-test-cases`;
- один existing TMS key нельзя размножать на несколько cases.

## Effective test set

Для coverage validation итоговое состояние вычисляется как:

`current existing test corpus + approved test-case-updates patches + approved new-test-cases`.

Checklist не считается independent coverage.

## TMS identity

- existing TMS key сохраняется;
- новый key не придумывается;
- UPDATE без подтверждённой identity блокируется;
- approved new case content и approved patch semantics не переписываются export stage.

## Existing-case discovery boundary

Existing test case discovery принадлежит `test-model-impact-analysis`, а не `test-case-maintainer`.

Обязательный upstream chain для UPDATE:

`test-model row → exact Zephyr/TMS key → actual current case retrieved → approved UPDATE intent → maintainer patch`.

Maintainer не должен заменять отсутствующий upstream baseline semantic search по похожему title.
