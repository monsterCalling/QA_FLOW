# QA Context Pack

Постоянный QA-контекст, который stages читают выборочно через `context-catalog.yaml`.

Основные файлы:

- `qa-strategy.md`;
- `testing-rules.md`;
- `test-design-techniques.md`;
- `priority-policy.md`;
- `risk-model.md`;
- `regression-rules.md`;
- `automation-rules.md`;
- `test-data.md`;
- `environment.md`;
- `known-quality-risks.md`;
- `test-model.md`;
- `test-cases/` — project-specific contract для manual test cases и TMS mapping.

## Test Case Contract

`test-cases/test-case-profile.yaml` определяет обязательные/необязательные поля, expected-result model, steps, test data и atomicity. Writer не должен угадывать эти правила.

`test-cases/test-case-examples.md` должен содержать 5–10 утверждённых примеров команды. Примеры задают стиль, но не business behavior текущего change.

`tms-field-mapping.yaml` задаёт mapping canonical `new-test-cases.json` / `test-case-updates.json` в поля внешнего IDE TMS plugin.

## Checklist

`checklist.md/json` — change-specific artifact. Markdown ориентирован на любого члена команды; technical traceability хранится преимущественно в JSON.

## Anti-Hallucination

QA baseline и examples не создают business rules. При конфликте приоритет имеют подтверждённые requirements, project-specific facts и USER_CONFIRMED Change Context.
