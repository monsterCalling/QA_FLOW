---
name: context-discovery
description: "Строит минимальный релевантный Context Plan для current OpenSpec change и следующих QA stages; не выполняет QA-анализ вместо downstream skills."
priority: 20
---

## Обязательные общие правила

Перед выполнением используй shared contracts, перечисленные в stage context, подготовленном `qa-flow` meta-skill. Если их содержимое уже включено в текущий контекст, не перечитывай файлы целиком:

- `qa-framework/shared/RULES.md`;
- `qa-framework/shared/CONTEXT_CONTRACT.md`;
- `qa-framework/shared/CLARIFICATIONS.md`;
- `qa-framework/shared/ARTIFACT_CONTRACT.md`;
- `qa-framework/shared/PROGRESS_EVENTS.md`.

Если `qa-flow` подготовил конкретные пути, ID, anchors, selectors или `context-manifest.json` — используй их. Не угадывай отсутствующие пути, Zephyr keys, topic names, table names, endpoint names или business rules.

### Context access policy

1. Не сканируй весь repository или весь Context Pack «на всякий случай».
2. Начинай с current change, approved upstream artifacts и context manifest.
3. Для больших источников сначала ищи по component/symbol/endpoint/topic/table/requirement ID, затем читай только релевантный диапазон.
4. Расширяй контекст только если имеющихся evidence недостаточно.
5. Если skill содержит `references/`, читай только те reference-файлы, которые релевантны текущему change.
6. Не считай отсутствие найденного текста доказательством отсутствия поведения в системе, пока поиск не был достаточным и воспроизводимым.

### Evidence policy

Каждый существенный вывод маркируй как один из:

- `FACT` — прямо подтверждён источником;
- `INFERENCE` — логический вывод из подтверждённых фактов;
- `ASSUMPTION` — неподтверждённое предположение;
- `UNKNOWN` — данных недостаточно.

Для finding/risk/mismatch/test expectation указывай минимально достаточный evidence: requirement/section, artifact ID, code symbol/file, test ID, schema/contract или другой источник.

`ASSUMPTION` нельзя превращать в обязательное expected behavior, defect или final verdict без подтверждения человека.

### Interactive rule

Если неизвестный факт способен существенно изменить severity, expected behavior, test design, implementation conclusion или verdict:

1. останови финализацию;
2. верни `WAITING_INPUT`;
3. задай минимальный набор targeted questions;
4. объясни, какое решение зависит от каждого ответа;
5. после подтверждения используй ответ как Change Context и не задавай тот же вопрос повторно.

# Context Discovery

## Роль

Ты — context router. Твоя задача — определить **какие источники и какие фрагменты** нужны downstream QA skills, а не выполнить spec review, risk analysis или test design самостоятельно.

## Цель

Сформировать минимальный, объяснимый и расширяемый context manifest, который уменьшает лишнее чтение и при этом не скрывает потенциально важный контекст.

## Не входит в scope

- не формировать QA findings;
- не назначать risk severity;
- не проектировать тесты;
- не утверждать implementation impact как факт без evidence;
- не читать весь repository рекурсивно.

## Входы

Приоритетно используй:

1. обязательные артефакты current OpenSpec change: `proposal.md`, `preview.md`, `spec.md` или `specs/**`, плюс явно связанные документы;
2. Change Context / confirmed clarifications;
3. индекс/структуру Project Context;
4. индекс/структуру QA Context;
5. test model index/anchors, если доступны;
6. список предыдущих artifacts текущего run;
7. repository search только для определения релевантных областей.

## Входной контракт

Для нового QA-flow сначала проверь наличие:

- `proposal.md`;
- `preview.md`;
- `spec.md` **или** каталога `specs/` с одной или несколькими спецификациями.

Если обязательного артефакта нет, верни `WAITING_INPUT`/`BLOCKED` и перечисли отсутствующие файлы. Не подменяй их догадками из кода.

Опциональные входы: DB schema/DDL, OpenAPI/Swagger, Kafka schema/contracts, diagrams, mappings, examples, `design.md`, `tasks.md`, ссылки на authoritative project docs.

## Алгоритм

### Шаг 1. Определи semantic scope change

Из change извлеки:

- business domain;
- actors/roles;
- business entities;
- operations/use cases;
- statuses/state transitions;
- API endpoints/contracts;
- Kafka events/topics;
- DB entities/tables/schema areas;
- integrations;
- configuration/feature flags;
- security/permissions;
- calculations/time rules;
- explicitly mentioned regression areas.

Не считай найденное имя компонента доказательством реального impact — на этом этапе это retrieval hint.

### Шаг 2. Сформируй retrieval keys

Для каждого элемента scope сформируй набор точных ключей:

- requirement ID;
- canonical business term;
- API path/method;
- event/topic/schema name;
- entity/table name;
- class/package/service name, если известен;
- Zephyr/test-model anchor;
- synonyms только если они подтверждены glossary/context.

### Шаг 3. Выбери QA Context

Подключай QA rules по признакам change:

- диапазоны/валидация → testing rules + test-design techniques;
- состояния → state-transition rules;
- Kafka → Kafka testing rules;
- DB/data lifecycle → DB/test-data rules;
- high-risk area → risk model;
- regression/shared change → regression rules;
- automation stage → automation rules.

Не добавляй все QA-файлы автоматически.

### Шаг 4. Выбери Project Context

Для каждого выбранного source укажи:

- зачем он нужен;
- какой scope/section/anchor читать;
- evidence, почему source релевантен;
- priority: REQUIRED / RECOMMENDED / ON_DEMAND.

### Шаг 5. Выбери previous artifacts

Подключай только upstream artifacts, способные изменить текущий stage. Например:

- risk-analysis не требует будущих test artifacts;
- test-design требует approved risk + test-model-impact;
- spec-vs-code требует approved spec + implementation context;
- qa-verdict требует validated final artifacts/results.

### Шаг 6. Оцени пробелы

Зафиксируй:

- missing project context;
- ambiguous mappings;
- unavailable authoritative source;
- stale source;
- conflicting sources.

Если без пробела downstream stage не сможет отличить FACT от ASSUMPTION — подготовь clarification.

### Шаг 7. Установи budget/expansion strategy

Для каждого большого source укажи preferred access:

- `SECTION`;
- `ANCHOR`;
- `SEARCH_THEN_RANGE`;
- `SYMBOL`;
- `FULL_FILE` — только для небольших обязательных файлов.

## Output requirements

`data` должен содержать как минимум:

- `scope_summary`;
- `retrieval_keys`;
- `required_sources`;
- `recommended_sources`;
- `on_demand_sources`;
- `previous_artifacts`;
- `helper_skills`;
- `missing_context`;
- `clarification_candidates`;
- `context_budget_notes`.

Для каждого source желательно указывать `path`, `selector`, `reason`, `evidence`, `priority`.

## Helper skill routing

Рекомендуй helper только при наличии сигнала:

- SQL/data relations → `sql-analysis`;
- Kafka/async → `kafka-analysis`;
- REST/OpenAPI → `api-analysis`;
- logs/diagnostics → `log-analysis`;
- сложные data prerequisites → `test-data-analysis`.

## Self-review

Перед `COMPLETED` проверь:

- нет ли blanket-selection «всех файлов»;
- current change покрыт всеми ключевыми domain areas;
- каждый REQUIRED source имеет причину;
- missing context не скрыт предположением;
- downstream skill сможет начать работу без повторного общего сканирования.

## Anti-patterns

- читать весь repo;
- возвращать список каталогов без selectors/reason;
- проводить spec-review внутри context-discovery;
- рекомендовать helper skill просто потому, что он существует;
- объявлять component affected только по совпадению слова.

## Definition of Done

Stage завершён, когда сформирован минимальный context plan, объяснены причины выбора источников и явно отмечены неизвестные области, требующие последующего расширения или clarification.
