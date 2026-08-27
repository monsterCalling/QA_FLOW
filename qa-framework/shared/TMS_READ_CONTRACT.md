# Existing Test Case Read Contract

## Purpose

Этот контракт описывает read-only handoff между QA Flow и доступной Jira/Zephyr integration для анализа **существующих** test cases.

Он не определяет конкретный vendor/API/MCP command. Runtime использует доступную configured integration.

## Input

Для чтения existing cases stage передаёт стабильные keys, найденные в текущем `test-model.md`:

```text
keys: [QA-T123, QA-T187, QA-T201]
```

По возможности keys читаются batch-операцией.

## Required returned content

Для каждого key желательно получить максимально актуальные поля:

- `key` / id;
- title/name;
- status;
- priority, если есть;
- preconditions;
- test data;
- ordered steps;
- per-step expected result, если используется;
- final expected result, если используется;
- labels/components/custom fields, если релевантны;
- requirement/test links, если доступны;
- revision/updated timestamp, если integration их отдаёт.

Не требуется, чтобы TMS имела именно такие названия полей: runtime приводит доступные данные к этой логической модели без изменения смысла.

## Retrieval statuses

Для каждого key фиксируй:

- `RETRIEVED` — содержимое получено;
- `NOT_FOUND` — key не найден;
- `ERROR` — integration вернула ошибку;
- `UNAVAILABLE` — read integration недоступна.

## Safety rules

1. Read stage не изменяет test case в TMS.
2. Не считать matrix row или title эквивалентом полного test case body.
3. Не классифицировать material case как `UPDATE/KEEP/OBSOLETE` без достаточного evidence.
4. Если read failed, сохранить exact key и error; не подменять test content предположением.
5. TMS content, полученный по exact key, считается текущим baseline для последующего `test-case-maintainer`.
6. `test-case-maintainer` не должен заново искать другой case по похожему title вместо переданного exact key.

## Recommended normalized record

```json
{
  "tms_key": "QA-T187",
  "retrieval_status": "RETRIEVED",
  "revision": "optional",
  "updated_at": "optional",
  "case": {
    "title": "...",
    "preconditions": [],
    "test_data": [],
    "steps": [],
    "final_expected_result": null,
    "priority": null,
    "links": []
  }
}
```

## Failure handling

Если хотя бы один HIGH-confidence/material candidate не удалось прочитать и без него нельзя безопасно завершить impact analysis:

`WAITING_INPUT: EXISTING_TEST_CASE_RETRIEVAL_REQUIRED`

Если configured read integration отсутствует:

`WAITING_INPUT: EXISTING_TEST_CASE_RETRIEVAL_UNAVAILABLE`

Пользователь может устранить integration issue или явно разрешить иной источник evidence. Flow не должен молча продолжать по одной test-model row.

## Persisted snapshot

После успешного read сохрани baseline в `.agent-runs/<change-id>/artifacts/existing-test-cases-snapshot.json` (или approved artifact path текущего runtime).

Downstream `test-case-maintainer` использует этот snapshot для `before` values. Если перед применением patch внешний plugin видит более новую TMS revision/current value, он возвращает conflict вместо silent overwrite.
