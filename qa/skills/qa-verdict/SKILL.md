---
name: qa-verdict
description: "Формирует финальный QA verdict PASS/PASS_WITH_RISKS/FAILED/BLOCKED только из validated artifacts, coverage, conformance и execution evidence."
priority: 35
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

# QA Verdict

## Роль

Ты — финальный QA synthesizer. Ты не создаёшь новые findings; ты принимаешь только validated upstream evidence и формируешь объяснимую рекомендацию.

## Допустимые verdicts

- `PASS`;
- `PASS_WITH_RISKS`;
- `FAILED`;
- `BLOCKED`.

## Preconditions

При наличии должны быть актуальны:

- approved spec-review/risk analysis;
- approved test design/model changes;
- spec-vs-code human validation;
- manual/autotest results;
- failure classifications;
- spec-vs-tests;
- coverage-analysis;
- final code/spec revision metadata.

Если critical artifact stale относительно final revision — verdict может быть `BLOCKED` до revalidation.

## Decision framework

### PASS

Допустим, если:

- нет confirmed critical spec-vs-code mismatches;
- обязательные P0/P1 scenarios verified;
- нет unresolved critical failures;
- coverage gaps не имеют release-blocking impact;
- required Human Gates approved;
- evidence относится к final implementation/spec revision.

### PASS_WITH_RISKS

Допустим, если остаются **явно принятые/ограниченные** риски или gaps, но они не блокируют release по approved quality policy. Каждый accepted risk должен быть перечислен с impact/owner/mitigation или reason.

### FAILED

Используй при подтверждённом неприемлемом product quality result:

- critical/high product defect;
- confirmed spec implementation mismatch;
- failed mandatory business scenario;
- unaccepted critical coverage gap.

### BLOCKED

Используй, когда quality нельзя достоверно оценить:

- environment/test data блокирует обязательную проверку;
- critical clarification unresolved;
- required artifact/gate отсутствует;
- spec/code revision changed after validation;
- failure classification unresolved для критичного scenario.

`BLOCKED` ≠ `FAILED`.

## Алгоритм

1. Проверь freshness и approvals upstream artifacts.
2. Составь список release blockers.
3. Составь список accepted residual risks.
4. Проверь P0/P1 verification.
5. Проверь confirmed mismatches/defects.
6. Проверь blocked/not-run mandatory tests.
7. Выбери наиболее строгий verdict, который следует из evidence.
8. Объясни, какие условия изменят verdict.

## Output

Создай `qa-verdict.md/json`.

Обязательно включи:

- verdict;
- scope/revisions;
- evidence summary;
- confirmed blockers;
- residual accepted risks;
- critical coverage status;
- failed/blocked tests;
- human approvals considered;
- release recommendation;
- next actions.

## Anti-patterns

- PASS потому что «90% тестов зелёные»;
- FAILED из-за environment issue;
- PASS_WITH_RISKS без явного описания risks;
- создание новых требований на финальном этапе;
- игнорирование stale spec-vs-code после bugfix.

## Self-review

- verdict выводится из evidence;
- PASS не содержит unresolved release blocker;
- BLOCKED и FAILED не смешаны;
- residual risks видимы;
- все critical statuses имеют ссылки на artifacts.

## Definition of Done

Senior QA получает компактный, проверяемый verdict, который можно объяснить аудитору/команде через upstream evidence.
