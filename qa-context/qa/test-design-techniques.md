# Test Design Techniques

## 1. Назначение

Этот документ является обязательным QA Context для `test-design` skill.

Он задаёт техники тест-дизайна, правила их выбора и минимальные ожидания к покрытию. Агент обязан **выбрать релевантные техники и объяснить выбор**, а не механически применять все техники к каждому requirement.

Техника помогает системно получить test conditions. Она **не создаёт новое бизнес-поведение**. Expected result всегда должен происходить из подтверждённой спецификации, project context или human clarification.

## 2. Обязательное правило выбора

Для каждого нового/изменённого requirement или business rule агент должен:

1. определить тип поведения;
2. выбрать одну или несколько релевантных техник;
3. указать причину выбора;
4. получить test conditions из техники;
5. связать test conditions с requirement и risk;
6. отметить неприменимые очевидные техники, если их отсутствие может вызвать вопрос на review;
7. запросить clarification, если техника требует неизвестного expected behavior.

Не допускается формулировка `использованы стандартные техники тест-дизайна` без перечисления конкретных техник.

## 3. Equivalence Partitioning — классы эквивалентности

### Когда применять

Использовать, когда входные данные можно разделить на группы, внутри которых система должна вести себя одинаково.

Типичные признаки:

- enum/category/type;
- диапазон значений;
- валидный/невалидный формат;
- разные типы пользователей/продуктов/операций;
- optional/required data;
- состояния сущности с одинаковым ожидаемым поведением.

### Как проектировать

Выделить:

- valid classes;
- invalid classes;
- специальные классы, если для них есть отдельное поведение.

Из каждого значимого класса выбрать представителя.

### Пример

Requirement: возраст клиента допускается `18..65`.

Классы:

- `<18` — invalid;
- `18..65` — valid;
- `>65` — invalid.

Equivalence Partitioning не заменяет Boundary Value Analysis: для границ применяются обе техники.

## 4. Boundary Value Analysis — анализ граничных значений

### Когда применять

Использовать при:

- min/max;
- length;
- date/time boundary;
- amount/limit/rate;
- pagination limits;
- collection size;
- threshold business rule.

### Базовый набор

Для диапазона `[min, max]` рассмотреть минимум:

- `min - 1`;
- `min`;
- `min + 1`;
- nominal value;
- `max - 1`;
- `max`;
- `max + 1`.

Для строк/коллекций аналогично использовать длину/размер.

Если домен дискретный или соседнее значение невозможно, выбрать ближайшее допустимое значение и объяснить это.

## 5. Decision Table Testing — таблицы решений

### Когда применять

Использовать, когда результат зависит от комбинации нескольких условий.

Типичные признаки:

- несколько boolean/enum условий;
- eligibility;
- permissions;
- tariff/product rules;
- маршрутизация;
- комбинация статуса, типа, роли, суммы и т.п.

### Как проектировать

1. выписать conditions;
2. выписать actions/results;
3. построить meaningful combinations;
4. исключить невозможные комбинации только при наличии подтверждённого ограничения;
5. получить минимум один test condition на каждое значимое правило таблицы.

Если комбинаций много — после таблицы решений допустимо применить pairwise к независимым параметрам.

## 6. State Transition Testing — переходы состояний

### Когда применять

Обязательно рассмотреть, если change затрагивает:

- status;
- lifecycle;
- state machine;
- workflow;
- terminal state;
- последовательность событий.

### Покрытие

Проверить:

- разрешённые переходы;
- запрещённые переходы;
- переход в то же состояние;
- переход из terminal state;
- переход при missing precondition;
- повторную команду;
- конкурентные переходы, если возможны;
- side effects перехода;
- history/audit;
- API/DB/Kafka consistency.

Если доступна таблица переходов, test conditions должны быть traceable к её рёбрам.

## 7. Use Case / Scenario Testing — бизнес-сценарии

### Когда применять

Использовать для end-to-end или business flow, состоящего из нескольких шагов и участников.

Покрывать:

- основной успешный поток;
- альтернативные потоки;
- ошибки на каждом критичном шаге;
- прерывание/повторное продолжение;
- side effects;
- состояние после частичной ошибки.

Не заменять этой техникой точечный BVA/Decision Table/State Transition, если они нужны для отдельных правил.

## 8. Cause–Effect / Rule Dependency Analysis

### Когда применять

Когда несколько входных причин приводят к одному или нескольким эффектам и зависимости сложнее простой валидации.

Использовать для:

- business eligibility;
- routing;
- feature flags;
- зависимых validation rules;
- complex integration decision.

Агент должен явно записать связь `cause → effect` и затем получить test conditions.

## 9. Pairwise / Combinatorial Testing

### Когда применять

Использовать, если:

- есть несколько относительно независимых параметров;
- полный Cartesian product слишком велик;
- нет requirement, требующего exhaustive coverage всех комбинаций.

### Правила

- pairwise применяется после выделения валидных значений/классов;
- критичные business combinations добавляются отдельно, даже если pairwise их не выбрал;
- P0/P1 combination нельзя исключать только из-за сокращения pairwise;
- запрещено применять pairwise к зависимым параметрам без учёта constraints.

## 10. Error Guessing / Defect-Based Testing

### Когда применять

Использовать как дополнительную технику на основе:

- известных дефектов;
- `known-quality-risks.md`;
- исторически проблемных областей;
- особенностей технологии;
- опыта QA.

Типовые идеи:

- duplicate;
- retry after timeout;
- stale data;
- race condition;
- out-of-order event;
- malformed payload;
- partial failure;
- null/empty/whitespace;
- precision/rounding;
- wrong timezone.

Error Guessing **не позволяет придумывать expected behavior**. Если ожидаемый результат неизвестен — clarification.

## 11. Negative Testing

Negative testing — это не одна отдельная техника вместо остальных, а обязательная перспектива для значимых input/state/integration rules.

Рассмотреть:

- invalid input;
- missing input;
- unauthorized action;
- forbidden state;
- unavailable dependency;
- timeout;
- malformed event;
- duplicate command/event;
- unsupported value;
- invalid transition.

Negative scenario должен иметь подтверждённый expected behavior или clarification.

## 12. CRUD / Data Lifecycle Testing

### Когда применять

Если change меняет жизненный цикл persistent entity.

Рассмотреть:

- create;
- read;
- update;
- delete/soft delete;
- restore, если есть;
- history/versioning;
- repeated create/update;
- relations;
- referential integrity;
- visibility after state change.

## 13. Idempotency Testing

### Когда применять

Обязательно рассмотреть для:

- повторяемых commands;
- POST/operation API, если есть idempotency contract;
- Kafka consumers;
- callbacks;
- retries;
- scheduled/batch processing.

Проверить:

- same idempotency/event key;
- повтор до завершения первого вызова;
- повтор после success;
- повтор после timeout/unknown result;
- side effects count;
- DB records count;
- outgoing event count.

Если наличие идемпотентности не подтверждено — агент должен спросить, а не объявлять её requirement.

## 14. Concurrency / Race Condition Testing

### Когда применять

Если две операции могут выполняться одновременно над общей сущностью/ресурсом.

Рассмотреть:

- two identical commands;
- conflicting updates;
- update + status transition;
- duplicate events processed concurrently;
- optimistic/pessimistic locking;
- lost update;
- double side effect.

Проверять final business state, а не только ответы отдельных запросов.

## 15. Retry / Recovery Testing

### Когда применять

Для integration/async flows с retry/recovery semantics.

Рассмотреть:

- first attempt fail → retry success;
- all retries fail;
- timeout with unknown downstream result;
- process/service restart between attempts;
- duplicate side effect during retry;
- exhausted retry → DLQ/error state;
- recovery after dependency becomes available.

## 16. Contract Testing Perspective

При изменении REST/Kafka/external integration contract рассмотреть:

- required/optional fields;
- type changes;
- enum changes;
- backward compatibility;
- old producer/new consumer;
- new producer/old consumer, если применимо;
- unknown fields;
- missing fields;
- serialization/deserialization.

Это может сочетаться с Equivalence Partitioning и Decision Tables.

## 17. Security-Oriented Test Design

Если change затрагивает authentication/authorization/data access, рассмотреть:

- unauthenticated;
- expired/invalid credential;
- insufficient role;
- cross-user/cross-tenant object access;
- privilege escalation;
- sensitive data exposure;
- unsafe logging.

Если security behaviour не описано и не содержится в project rules — clarification.

## 18. Technique Selection Matrix

| Характеристика requirement | Основные техники |
|---|---|
| Диапазон, min/max, length | Equivalence Partitioning + Boundary Value Analysis |
| Несколько условий влияют на результат | Decision Table / Cause–Effect |
| Статусы/lifecycle | State Transition Testing |
| Много независимых параметров | Equivalence Partitioning + Pairwise |
| Длинный business flow | Use Case / Scenario Testing |
| Persistent entity lifecycle | CRUD / Data Lifecycle |
| Повторная команда/event | Idempotency + Error Guessing |
| Async/retry | Retry/Recovery + Idempotency + Scenario Testing |
| Возможны параллельные изменения | Concurrency / Race Condition |
| REST/Kafka contract | Contract perspective + EP/BVA для полей |
| Permissions/security | Decision Table + Security-oriented design |
| Исторически дефектная область | Error Guessing дополнительно |

Матрица — рекомендация, а не автоматическая подстановка.

## 19. Минимальный алгоритм `test-design` агента

Для каждого requirement:

```text
Requirement
    ↓
Определить business rule / observable behavior
    ↓
Определить risk
    ↓
Выбрать техники
    ↓
Обосновать выбор
    ↓
Сформировать test conditions
    ↓
Проверить negative/boundary/state/integration аспекты
    ↓
Связать verification points
    ↓
Определить manual need и automation candidacy независимо
```

## 20. Обязательный output для Human Review

Для каждого нового/изменённого test condition показать:

- source requirement;
- source risk;
- applied technique(s);
- rationale;
- test condition;
- expected behavior;
- test data needs;
- verification points;
- manual required;
- automation candidate;
- unresolved assumptions/unknowns.

Senior QA должен иметь возможность понять **почему этот тест появился** без повторного проектирования покрытия с нуля.
