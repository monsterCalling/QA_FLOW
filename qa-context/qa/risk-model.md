# Risk Model

## 1. Назначение

Документ задаёт базовую модель risk-based testing. Правила остановки workflow определяются отдельно в `priority-policy.md`.

Каждый risk оценивается минимум по:

- failure mode;
- impact;
- probability/confidence;
- affected scope;
- reversibility/recoverability;
- business criticality;
- existing controls;
- test obligation.

## 2. Канонические уровни

Новые artifacts используют:

`CRITICAL / HIGH / MEDIUM / LOW / TRIVIAL`.

Для совместимости со старыми материалами:

- `P0 = CRITICAL`
- `P1 = HIGH`
- `P2 = MEDIUM`
- `P3 = LOW`

## 3. CRITICAL

Потенциальное последствие:

- двойное проведение/потеря критичной финансовой операции;
- необратимая порча критичных данных;
- нарушение финансовой целостности;
- массовая неконсистентность;
- критичная утечка/нарушение безопасности;
- невозможность выполнения ключевого бизнес-процесса;
- массовый отказ;
- неконтролируемое повторное выполнение критичной операции;
- отсутствие реалистичного recovery path.

Требования QA:

- Human Review риска;
- `test_obligation = MANDATORY`;
- отдельные failure/recovery/idempotency/consistency проверки по применимости;
- явная traceability risk → test condition → result.

Сам факт CRITICAL risk не означает requirement defect и не является автоматическим HARD_BLOCK.

## 4. HIGH

Потенциальное последствие:

- нарушение важного business flow;
- неверное состояние сущности;
- потеря/дублирование значимого integration event;
- серьёзная integration failure;
- нарушение idempotency;
- breaking backward compatibility;
- существенная business validation error;
- большой regression impact;
- заметный security/data consistency impact.

Требования QA:

- Human Review риска;
- `test_obligation = MANDATORY`;
- automation предпочтительна для стабильного повторяемого regression scenario.

## 5. MEDIUM

Потенциальное последствие:

- локальная ошибка сценария;
- ограниченный customer/user impact;
- recoverable integration/data issue;
- существенная validation/error-handling проблема без критичного эффекта;
- проблема с доступным workaround.

Требования QA:

- Human Review;
- `test_obligation = RECOMMENDED`.

## 6. LOW

Потенциальное последствие:

- низкий business impact;
- легко обнаружимая/recoverable проблема;
- minor behavior/observability issue без влияния на основной flow.

Требования QA:

- flow автоматически продолжается;
- `test_obligation = OPTIONAL`.

## 7. TRIVIAL

Практически отсутствует business impact. Обычно отдельный risk создавать не нужно; категория допустима, если наблюдение нужно сохранить для traceability.

Требования QA:

- flow автоматически продолжается;
- `test_obligation = NONE`.

## 8. Факторы повышения impact

Рассмотреть особенно внимательно, если change затрагивает:

- деньги/расчёты;
- статусы/state machine;
- authorization/security;
- персональные/чувствительные данные;
- shared component/library;
- Kafka contract;
- DB schema/migration;
- transaction boundaries;
- retry/idempotency;
- concurrency;
- batch processing;
- external integration;
- несколько сервисов;
- backward compatibility.

Фактор сам по себе не определяет priority: требуется конкретный failure mode и consequence.

## 9. Факторы probability

Probability может быть выше, если:

- логика новая/сложная;
- change большой;
- требования неоднозначны;
- отсутствует existing coverage;
- много integration points;
- async processing;
- сложная state machine;
- исторически проблемная область;
- legacy/shared code;
- слабая observability;
- сложные test data/migration interactions.

Если probability нельзя оценить по evidence — укажи `UNKNOWN`, не придумывай число.

## 10. Controls и residual risk

Для каждого значимого risk учитывай существующие controls:

- constraints;
- validation;
- idempotency storage;
- retries;
- transactions;
- monitoring;
- existing tests.

Control снижает concern только при наличии evidence; его существование не доказывает корректность.

## 11. Risk vs Finding

Не смешивай:

- **Finding**: проблема/пробел/противоречие в требованиях или аналитических artifacts.
- **Risk**: потенциальный failure mode реализации/продукта.

Пример:

- `HIGH risk`: duplicate Kafka event может вызвать повторный side effect — это test focus.
- `HIGH finding`: spec не определяет, как повторный event должен обрабатываться, хотя это меняет expected behavior — это requirement problem.

Если risk-analysis обнаружил второе, создай `finding_candidate` и предложи возврат в `spec-review` вместо автоматического HARD_BLOCK на основании самого risk.

## 12. Test obligation

- `CRITICAL` → `MANDATORY`
- `HIGH` → `MANDATORY`
- `MEDIUM` → `RECOMMENDED`
- `LOW` → `OPTIONAL`
- `TRIVIAL` → `NONE`

`test-design` обязан обеспечить traceability для всех confirmed `MANDATORY` risks.

## 13. Правило для агента

Не назначай высокий priority только потому, что приложение банковское.

Priority определяется конкретным возможным последствием и evidence.

Если business impact неизвестен и это может изменить priority/gate — верни blocking clarification согласно `priority-policy.md`.
