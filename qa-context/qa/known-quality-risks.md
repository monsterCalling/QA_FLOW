# Known Quality Risks

## 1. Назначение

Этот файл содержит базовый каталог типовых quality risks.

Конкретные исторические риски проекта должны добавляться отдельными пунктами с источником.

## 2. Data Consistency

Типовые риски:

- partial update;
- данные изменились в одной системе и не изменились в другой;
- duplicate records;
- stale state;
- потеря history;
- неверная version;
- transaction partially committed.

## 3. Idempotency

- повторный REST request;
- duplicate Kafka message;
- retry после timeout;
- повторный callback;
- consumer restart;
- повторное выполнение batch item.

Последствие: duplicate side effect.

## 4. Async Processing

- eventual consistency;
- race condition;
- out-of-order;
- delayed processing;
- потерянный event;
- duplicate event;
- processing after entity state changed.

## 5. Integration

- contract drift;
- неизвестный enum;
- missing field;
- timeout;
- retry storm;
- malformed response;
- partial failure;
- breaking change.

## 6. State Machines

- запрещённый переход;
- переход из terminal state;
- race между двумя переходами;
- API разрешает то, что запрещает business rule;
- history не соответствует current state.

## 7. Financial / Calculation

Если применимо:

- rounding;
- precision;
- duplicate charge/operation;
- incorrect sign;
- incorrect currency;
- wrong effective date;
- inconsistent calculation between systems.

## 8. Security

- broken authorization;
- IDOR;
- excessive permissions;
- sensitive data in logs;
- secrets in response;
- insecure defaults.

## 9. Regression

- shared component modified;
- contract changed silently;
- common DTO changed;
- migration broke old data;
- feature flag affects legacy flow.

## 10. Observability

- missing correlation ID;
- error logged without context;
- sensitive data in logs;
- swallowed exception;
- retry without clear diagnostic information.

## 11. Testability

- невозможно подготовить нужное состояние;
- невозможно наблюдать side effect;
- нет стабильного test endpoint;
- нестабильные stubs;
- нет способа отличить timeout от functional error.

## 12. Project-specific risks

Добавлять по шаблону:

```text
ID:
Risk:
Area:
Evidence/source:
Impact:
Known triggers:
Required regression:
Owner:
Last reviewed:
```
