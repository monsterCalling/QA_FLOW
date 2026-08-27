# Artifact Contract

## Runtime drafts

`.agent-runs/<change-id>/artifacts/<stage>/`

## Approved canonical QA artifacts

После Human Approve: `openspec/changes/<change-id>/qa/`.

Draft нельзя выдавать за approved artifact.

## Common JSON envelope

Каждый stage JSON содержит version/change/stage/status/generated_at/context/warnings/data и не содержит chain-of-thought.

## Checklist

- `checklist.md` — human-facing командный список «что проверить»;
- `checklist.json` — technical traceability и structured items.

Checklist не является independent coverage.

## New manual test cases

- `new-test-cases.json` — canonical source только новых cases;
- `new-test-cases.md` — human-review projection полных новых cases.

Новый case предназначен для CREATE и имеет `tms_key = null` до подтверждённого TMS response.

## Existing test case updates

- `test-case-updates.json` — canonical patch plan существующих cases;
- `test-case-updates.md` — человекочитаемый список «какой TC → что изменить → было/стало → почему».

Update artifact не должен содержать полный rewrite existing case, если change локальный.

## Pre-export

- `pre-export-validation.md/json` — integrity + traceability + effective coverage + TMS compatibility;
- `export-plan.json` — CREATE/UPDATE/SKIP/PROPOSE_OBSOLETE для внешнего IDE plugin;
- `tms-mapping.json` — local ID ↔ confirmed TMS key.

Export artifacts не имеют права менять approved new test semantics или approved update patch semantics.

## Existing test case candidate artifacts

`test-model-impact-analysis` создаёт отдельный intermediate candidate set до impact classification:

- `test-case-candidates.json` — canonical список existing Zephyr/TMS keys, найденных через current `test-model.md`, причины выбора, confidence и retrieval status;
- `test-case-candidates.md` — human-readable projection того же списка.

`test-case-candidates.*` не является update plan и не означает, что каждый найденный case нужно менять.

После candidate discovery stage обязан прочитать actual current test case по exact key через доступную Jira/Zephyr read integration. Только после inspection формируется `test-model-impact.md/json` с `KEEP / UPDATE / OBSOLETE / ADD / REVIEW`.

### Existing test case snapshot

`existing-test-cases-snapshot.json` — immutable machine-readable snapshot actual current TMS cases, успешно retrieved на `test-model-impact-analysis`.

Он нужен для resume и точного downstream patching. Snapshot содержит exact key, optional revision/updated timestamp, retrieval source и current normalized fields, но **не содержит proposed changes**.
