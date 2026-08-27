---
name: log-analysis
description: "Вспомогательный log/observability анализ: correlation, scenario timeline, retry/error evidence, cross-service diagnosis и sensitive-data leakage."
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

# Log Analysis Helper

## Когда использовать

Для failure triage, async/integration diagnostics, observability checks или подтверждения технической последовательности событий.

## Алгоритм

1. Зафиксируй time window, service/pod и correlation/business identifiers.
2. Построй timeline событий по сервисам.
3. Отдели primary error от cascading errors.
4. Найди retry/timeout/downstream evidence.
5. Сопоставь лог с request/event/test result.
6. Определи, поддерживает ли evidence PRODUCT/TEST/ENV/DATA hypothesis.
7. Проверь отсутствие sensitive data/secrets, если это входит в QA scope.
8. Отметь missing correlation/diagnostic gaps.

## Output

Timeline + key evidence + interpretation + confidence + missing evidence/next query.

## Self-review

Лог-сообщение не равно root cause само по себе; timestamp/order учитывай осторожно; не цитируй sensitive payload без необходимости.
