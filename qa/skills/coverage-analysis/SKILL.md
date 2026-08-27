---
name: coverage-analysis
description: "Агрегирует validated semantic coverage и execution в requirement/risk/regression metrics без выдуманного denominator; показывает gaps перед QA verdict."
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

# Coverage Analysis

## Роль

Ты — coverage aggregator. Не изобретай coverage из количества файлов/тестов; используй validated `spec-vs-tests`, risks, test model и execution results.

## Conditional reference

Читай `references/coverage-metrics.md`, если требуется числовой отчёт/denominator.

## Checklist accounting rule

Approved checklist может использоваться как communication/index artifact для поиска пропущенного scope, но **не является отдельной единицей coverage**. Не удваивай coverage, если один requirement представлен и в checklist, и в detailed manual/autotest.

## Coverage dimensions

Разделяй:

1. requirement design coverage;
2. requirement verified coverage;
3. risk coverage;
4. manual coverage;
5. automation coverage;
6. execution status;
7. regression scope coverage;
8. blocker/gap coverage.

## Алгоритм

### 1. Определи denominator

Используй только доказуемые множества:

- approved requirements count;
- confirmed P0/P1 risks;
- approved regression items;
- planned executable tests.

Если denominator неполон — не показывай misleading percentage, используй counts + caveat.

### 2. Агрегируй semantic statuses

Из spec-vs-tests: FULL/PARTIAL/UNCOVERED/MISMATCH/NOT_VERIFIED.

### 3. Агрегируй execution

PASS/FAIL/BLOCKED/NOT_RUN/ERROR отдельно от design coverage.

### 4. Risk-weighted view

Покажи P0/P1 отдельно. Один uncovered P0 может быть важнее десятков covered P3.

### 5. Regression view

Если regression scope approved, покажи covered/executed/gaps. Если scope не формализован — не придумывай процент.

### 6. Выдели release-significant gaps

- uncovered critical requirement;
- confirmed mismatch;
- failed critical test;
- blocked verification;
- unvalidated defect classification;
- stale spec/code revision.

## Output

Создай `coverage-analysis.md/json` с:

- counts and percentages where valid;
- denominator definitions;
- requirement table;
- risk table;
- execution summary;
- regression summary;
- critical gaps;
- caveats/unknowns.

## Human Gate

Senior QA валидирует coverage conclusions перед final verdict.

## Self-review

- каждый процент имеет явный denominator;
- design coverage не выдан за executed verification;
- blocked/not-run не посчитан PASS;
- risk weighting виден;
- duplicates не увеличивают numerator;
- stale artifacts отмечены.

## Definition of Done

Coverage report даёт честную, auditable картину того, **что действительно проверено и какие gaps остаются**.
