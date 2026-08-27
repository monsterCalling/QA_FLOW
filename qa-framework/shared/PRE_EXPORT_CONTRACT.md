# Pre-Export Validation Contract

`pre-export-validation` выполняется только после Human Approve checklist, new test cases, update plan и approved test-model update.

## Inputs

- approved `new-test-cases.json`;
- approved `test-case-updates.json`;
- exact current content существующих cases, затронутых UPDATE;
- approved test-design / impact / test model;
- configured TMS field mapping.

## Effective test set

Coverage проверяется не по одному файлу, а по состоянию после применения change:

`existing test corpus + update patches + new test cases`.

### Integrity — new cases

- JSON parseable;
- local IDs уникальны;
- обязательные поля соответствуют profile;
- `tms_key = null` для CREATE;
- нет unresolved atomicity violations;
- MD и JSON относятся к одной approved версии.

### Integrity — updates

- exact identity каждого UPDATE подтверждена;
- existing TMS key не конфликтует;
- current existing body доступен;
- каждый patch имеет field/step locator, before/after, reason/source refs;
- `before` совпадает с current source snapshot;
- split requests реализованы в `new-test-cases` либо явно resolved;
- `PROPOSE_OBSOLETE` не превращается в delete.

### Traceability / coverage

После логического применения patches:

- каждый active approved test condition покрыт минимум одним effective manual test либо имеет approved exclusion;
- каждый requirement с test obligation связан с test condition и effective case;
- CRITICAL/HIGH risk с MANDATORY obligation покрыт;
- MEDIUM RECOMMENDED не теряется без documented decision;
- checklist не засчитывается как отдельное coverage.

### Export compatibility

- `tms-field-mapping.yaml.project_configured == true`;
- required CREATE fields mapped;
- fields/steps, указанные в UPDATE patch, поддерживаются plugin mapping;
- UPDATE без confirmed TMS identity → BLOCKED;
- REVIEW impact блокирует export plan.

## Result

- `pre-export-validation.md`;
- `pre-export-validation.json`;
- status `PASSED` или `BLOCKED`.

Только `PASSED` разрешает сформировать export plan.
