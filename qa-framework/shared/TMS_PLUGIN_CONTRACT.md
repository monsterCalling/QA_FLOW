# IDE TMS Plugin Handoff Contract

## Purpose

Внешний IDE plugin является executor фактической синхронизации с Zephyr/TMS. QA flow подготавливает lossless handoff и не перегенерирует approved test content.

## Inputs

Plugin рекомендуется принимать:

1. `new-test-cases.json` — approved full content для CREATE;
2. `test-case-updates.json` — approved exact patches для UPDATE / PROPOSE_OBSOLETE;
3. `export-plan.json` — набор операций и source pointers;
4. `qa-context/qa/test-cases/tms-field-mapping.yaml`;
5. `tms-mapping.json` — local ID ↔ TMS key, если есть.

## CREATE

CREATE разрешён только для items из `new-test-cases.json`. Plugin не должен создавать case из update artifact или checklist.

## UPDATE

UPDATE разрешён только при confirmed existing TMS key.

Для каждой patch operation plugin должен:

1. прочитать current target value;
2. сравнить его с `before`;
3. если отличается → вернуть `CONFLICT`, не применять silent overwrite;
4. применить `after` только при совпадении guard;
5. вернуть per-operation/per-case result.

Это защищает от обновления уже изменившегося TC устаревшим plan.

## Lossless mapping

Plugin не должен:

- сокращать steps/expected;
- объединять cases;
- семантически менять title/objective;
- терять required test data/preconditions;
- применять fields, которых нет в approved patch;
- создавать cases из checklist;
- превращать `PROPOSE_OBSOLETE` в delete.

Unsupported field/path → `MAPPING_ERROR`.

## Recommended result

```json
{
  "change_id": "CHANGE-ID",
  "results": [
    {
      "source": "new-test-cases.json",
      "local_id": "MT-001",
      "action": "CREATE",
      "status": "SUCCESS",
      "tms_key": "QA-T1234",
      "error": null
    },
    {
      "source": "test-case-updates.json",
      "update_id": "UPD-001",
      "local_id": "MT-010",
      "action": "UPDATE",
      "status": "SUCCESS",
      "tms_key": "QA-T1200",
      "error": null
    }
  ]
}
```

## Efficiency

- new case → CREATE;
- affected existing → UPDATE only listed fields/steps;
- unchanged → SKIP;
- obsolete → PROPOSE_OBSOLETE;
- optional content hash may be calculated deterministically by plugin/exporter, not LLM.

## Post-export checks

- requested CREATE/UPDATE count matches results;
- every successful CREATE has returned TMS key;
- every UPDATE targeted expected key;
- conflicts/errors remain unexported;
- mapping updates only on SUCCESS.
