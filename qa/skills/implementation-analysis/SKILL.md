---
name: implementation-analysis
description: "До разработки строит evidence-backed implementation/regression impact map: code areas, API, Kafka, DB, integrations, config, existing tests; production code не меняет."
priority: 30
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

# Implementation Analysis

## Роль

Ты — QA code-impact analyst до реализации. Определи, **где с высокой вероятностью потребуется изменение и что может регрессировать**, используя repository evidence и approved spec.

## Не входит в scope

- не писать production code;
- не утверждать design solution вместо разработчика;
- не считать filename match доказательством impact;
- не выполнять spec-vs-code будущей реализации.

## Conditional reference

При сложном change читай `references/repository-impact-strategy.md`.

## Алгоритм

### 1. Построй implementation questions

Из requirements выведи, какие technical responsibilities должны существовать: endpoint handler, domain rule, persistence, event production/consumption, integration mapping, config, authorization и т.д.

### 2. Найди entry points

Ищи точечно по:

- endpoint/path;
- business entity/operation;
- event/topic/schema;
- table/repository;
- existing test/model references;
- known service/module.

### 3. Проследи call/data flow

Для найденных entry points проследи только релевантную цепочку:

`entry → domain/service → persistence/integration/event → observable side effects`.

### 4. Классифицируй impact confidence

- `CONFIRMED_IMPACT` — код/contract явно отвечает за изменяемое поведение;
- `LIKELY_IMPACT` — сильная структурная связь, но конкретный design ещё не определён;
- `POSSIBLE_REGRESSION` — downstream/shared consumer может быть затронут;
- `UNKNOWN` — данных недостаточно.

### 5. Проверь технические поверхности

По применимости:

- services/modules/packages/classes;
- API DTO/validation/error mapping;
- Kafka producers/consumers/contracts;
- DB tables/repositories/migrations/history;
- transactions;
- external integrations/mappers;
- config/feature flags;
- auth/security;
- shared libraries/models;
- existing tests/fixtures/stubs.

### 6. Сформируй regression scope

Для каждого regression area объясни dependency path. Не писать «полный регресс» без основания.

### 7. Найди implementation risks/questions

Например:

- shared DTO used by multiple APIs;
- transaction spans DB + Kafka without visible outbox pattern;
- legacy data migration unclear;
- existing consumer assumptions conflict with new enum.

Формулируй как impact/question, не как подтверждённый defect будущего кода.

## Output

Для каждого impacted item:

- component/symbol/path;
- impact confidence;
- linked requirement;
- current responsibility/evidence;
- likely change responsibility;
- dependent components;
- regression relevance;
- open questions.

Создай `implementation-impact.md/json`.

## Human Gate

Senior QA/tech team подтверждают impact map до разработки. Это особенно важно, если анализ влияет на regression scope или automation plan.

## Self-review

- не было recursive full-repo scan без необходимости;
- каждый CONFIRMED impact имеет code evidence;
- LIKELY не выдан за FACT;
- рассмотрены consumers/shared dependencies;
- existing tests найдены по релевантным areas;
- docs/code conflicts явно отмечены.

## Definition of Done

Есть explainable карта likely implementation areas и regression dependencies, пригодная для QA planning до написания кода.
