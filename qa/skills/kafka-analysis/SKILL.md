---
name: kafka-analysis
description: "Вспомогательный Kafka QA анализ: producer/consumer/contracts, duplicates/idempotency, retry/redelivery, ordering, offsets/restart, DLQ и compatibility."
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

# Kafka Analysis Helper

## Когда использовать

При изменении producer/consumer/event schema/topic/async flow либо когда Kafka является значимым verification point/risk.

## Алгоритм

1. Зафиксируй producer, consumer, topic и business purpose.
2. Определи event trigger и expected side effect.
3. Проверь contract: key, headers, required/optional fields, enum/version/schema.
4. Определи delivery semantics из evidence; не предполагай at-least-once/exactly-once.
5. Рассмотри duplicate/redelivery и idempotency, если они могут изменить business outcome.
6. Рассмотри retry/backoff/error handling/DLQ только если реально существуют/требуются.
7. Проверяй ordering/out-of-order, когда business behavior зависит от порядка.
8. Для consumer restart/offset оцени возможность repeated/lost processing.
9. Проверь downstream failure/partial side effects.
10. Оцени backward compatibility старых producers/consumers/schema versions.

## Output

Для каждого meaningful point: requirement/risk, event flow, evidence, concern/test objective, unknowns.

## Self-review

Не придумывай гарантию брокера, DLQ или retry policy; различай platform semantics и application-level idempotency.
