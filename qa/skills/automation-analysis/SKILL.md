---
name: automation-analysis
description: "Планирует автоматизацию approved test conditions: candidate, layer, CREATE/UPDATE, assertions, fixtures/stubs/data, execution constraints; код не пишет."
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

# Automation Analysis

## Роль

Определи **какие approved test conditions стоит автоматизировать, на каком уровне и каким минимальным способом доказать behavior**.

## Обязательный контекст

Используй approved test-design, existing autotests/framework, `qa-context/qa/automation-rules.md` и implementation impact.

## Conditional reference

При неоднозначном выборе уровня/стоимости читай `references/automation-selection.md`.

## Не входит в scope

- не писать тестовый код;
- не перепроектировать requirement;
- не заменять manual coverage автоматически;
- не обещать automation там, где нет controllable environment/data.

## Алгоритм

### 1. Оцени automation value

Для каждого condition:

- regression frequency;
- business/risk criticality;
- determinism;
- setup cost;
- maintenance cost;
- observability;
- external dependency controllability;
- execution time.

Решение: `AUTOMATE / DEFER / MANUAL_ONLY / ALREADY_COVERED`.

### 2. Выбери минимальный достаточный layer

Рассмотри:

- component/integration;
- API;
- Kafka integration;
- DB integration;
- E2E.

Предпочитай более низкий layer, если он полностью доказывает requirement. Не выбирай E2E только потому, что он «реалистичнее».

### 3. Найди existing coverage

Определи:

- existing test to UPDATE;
- reusable helper/client/fixture;
- existing stub/mock pattern;
- duplicated scenario.

### 4. Определи assertions

План должен явно сказать, какие observable facts тест обязан доказать:

- response/business result;
- persisted state/history;
- produced/consumed event;
- absence of duplicate side effect;
- downstream call;
- failure/recovery behavior.

### 5. Определи data/environment needs

- unique IDs;
- setup/cleanup;
- stub behavior;
- async waiting;
- feature flags;
- required service availability.

### 6. Оцени flakiness risk

Отметь потенциальные источники:

- shared data;
- fixed sleeps;
- uncontrollable external system;
- eventual consistency without polling;
- order dependency;
- environment-specific timing.

### 7. Сформируй implementation plan

Для каждого automation item:

- CREATE / UPDATE;
- target test area/file candidate;
- layer;
- required helpers;
- assertions;
- data;
- mocks/stubs;
- cleanup;
- traceability;
- blockers.

## Decision rules

- P0/P1 repeatable regression scenario имеет высокий приоритет, но automation не обязательна, если недетерминирована/неуправляема.
- Не автоматизируй case только для процента coverage.
- Не используй mock, если он скрывает именно тот integration behavior, который требуется проверить.
- Manual и automation могут сосуществовать.

## Output

Создай `automation-plan.md/json` с per-condition decisions и summary priority/backlog.

## Human Gate

Plan должен быть approved до `autotest-writer`.

## Self-review

- каждый AUTOMATE item связан с approved condition;
- layer выбран обоснованно;
- assertions доказывают business outcome;
- reuse existing framework рассмотрен;
- flakiness/data cleanup учтены;
- MANUAL_ONLY/DEFER имеют reason.

## Definition of Done

Autotest-writer получает конкретный, реализуемый и поддерживаемый plan без необходимости заново решать стратегию покрытия.
