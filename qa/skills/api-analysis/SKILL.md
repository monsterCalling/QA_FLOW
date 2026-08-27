---
name: api-analysis
description: "Вспомогательный REST/API QA анализ: contracts, validation, errors, auth, idempotency, pagination, compatibility и OpenAPI/code consistency."
priority: 15
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

# API Analysis Helper

## Когда использовать

Только если current task затрагивает REST/HTTP/OpenAPI/API contract или API является observable verification point.

## Алгоритм

1. Зафиксируй endpoint/method/version.
2. Сравни authoritative contract sources и, если нужно, implementation.
3. Проверь request: path/query/header/body, required/optional/nullability/types/enum/format.
4. Проверь response: status, schema, business error, headers, side effects.
5. Проверь validation/boundaries из spec, не выдумывая новые limits.
6. Проверь auth/role/ownership, если применимо.
7. Проверь idempotency/retry semantics только при наличии requirement/context.
8. Для list APIs проверь pagination/filter/sort semantics, если они затронуты.
9. Оцени backward compatibility для existing consumers.
10. Зафиксируй gaps/conflicts с evidence.

## Output

Дай structured observations: endpoint, issue/verification objective, expected vs actual/unknown, evidence, severity/confidence и recommended QA action.

## Self-review

Не путай HTTP 200 с business success; не придумывай undocumented codes; не объявляй optional field breaking change без понимания consumer semantics.
