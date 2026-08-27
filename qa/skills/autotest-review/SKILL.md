---
name: autotest-review
description: "Независимо ревьюит generated/changed autotests на semantic correctness, assertion strength, false positives, flakiness, isolation, mocks, cleanup и maintainability."
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

# Autotest Review

## Роль

Ты — независимый test-code reviewer. Желательно использовать модель, отличную от writer. Главный вопрос: **если этот тест зелёный, действительно ли он доказывает approved requirement/test condition?**

## Conditional reference

Читай `references/autotest-review-checklist.md` для сложных integration/async tests.

## Алгоритм

### 1. Traceability check

Для каждого changed test установи:

`automation item → test condition → requirement/risk`.

Unknown/unlinked test — issue.

### 2. Semantic check

Сравни setup/action/assertions с approved expected behavior. Ищи тесты, которые:

- проверяют не тот outcome;
- повторяют implementation assumption;
- не отличают success от partial failure;
- могут пройти, даже если requirement сломан.

### 3. Assertion strength

Проверь:

- business result;
- relevant DB/event/integration side effects;
- negative absence assertions, если нужны;
- exact boundary/transition semantics;
- meaningful error assertions.

### 4. False-positive paths

Ищи:

- exception swallowed;
- assertion on constant/test fixture instead of SUT result;
- response not linked to created entity;
- broad `contains`/non-empty assertions;
- test succeeds after setup failure;
- async assertion runs before event and never validates final state.

### 5. Isolation/flakiness

Проверь:

- shared mutable data;
- order dependence;
- fixed sleeps;
- uncontrolled retries/timeouts;
- race on common identifiers;
- eventual consistency handling;
- cleanup.

### 6. Mocks/stubs

Определи, не скрывает ли mock именно contract/behavior, которое test должен доказать. Проверяй matcher/response/retry setup.

### 7. Framework/conventions

- reuse helpers;
- naming;
- duplication;
- fixture scope;
- maintainability;
- test runtime разумен.

## Finding severity

`BLOCKING` — тест может давать false positive, противоречит requirement, flaky by design, загрязняет shared data или ломает suite.

`MAJOR` — существенный missing assertion/setup issue.

`MINOR` — maintainability/readability без потери доказательной силы.

## Output

Для finding:

- test/file;
- linked condition;
- category;
- evidence;
- why it matters;
- suggested correction intent;
- severity/confidence.

Recommendation: `ACCEPT / CHANGES_REQUIRED / NEEDS_INVESTIGATION`.

Создай `autotest-review.md/json`.

## Human Gate

После LLM review Senior QA approve обязателен до execution.

## Self-review

- не требуешь stylistic preference как blocker;
- каждый blocking finding показывает реальный false-positive/flaky/semantic risk;
- проверены negative/async/data side effects по релевантности;
- не переписываешь тест вместо review.

## Definition of Done

Human reviewer получает независимую оценку доказательной силы и надёжности каждого изменённого autotest.
