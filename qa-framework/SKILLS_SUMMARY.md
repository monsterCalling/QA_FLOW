# QA FLOW FOR SDD v.1.0 — Skills Summary

Total skills: **28**.

| Skill | Type |
|---|---|
| `api-analysis` | helper |
| `automation-analysis` | stage |
| `autotest-review` | stage |
| `autotest-writer` | stage |
| `checklist-writer` | stage |
| `context-discovery` | stage |
| `coverage-analysis` | stage |
| `final-conformance-check` | stage |
| `implementation-analysis` | stage |
| `kafka-analysis` | helper |
| `log-analysis` | helper |
| `pre-export-validation` | stage |
| `qa-flow` | controller |
| `qa-verdict` | stage |
| `risk-analysis` | stage |
| `spec-review` | stage |
| `spec-review-critic` | stage |
| `spec-vs-code` | stage |
| `spec-vs-tests` | stage |
| `sql-analysis` | helper |
| `test-case-maintainer` | stage |
| `test-case-writer` | stage |
| `test-data-analysis` | helper |
| `test-design` | stage |
| `test-failure-analysis` | stage |
| `test-model-impact-analysis` | stage |
| `test-model-maintainer` | stage |
| `tms-export` | stage |

## Updated existing-case analysis

`test-model-impact-analysis` теперь создаёт `test-case-candidates.md/json`, отбирая existing candidates из текущего `test-model.md` прежде всего по полю `Тип проверки`, затем читает actual cases по exact Zephyr/TMS keys и только после inspection классифицирует impact. `test-case-maintainer` выполняет только точные patches approved existing cases.

`existing-test-cases-snapshot.json` фиксирует current retrieved baseline для exact `before → after` patching downstream.
