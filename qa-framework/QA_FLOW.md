# QA FLOW FOR SDD v.1.0

## Architecture

```text
User → GigaCode/LLM runtime → qa-flow meta-skill
                           ├→ qa-flow.yaml
                           ├→ state.json
                           ├→ selective context catalog
                           └→ current stage SKILL.md
```

## QA Design Flow

```text
proposal + preview + spec
        ↓
context-discovery
        ↓
spec-review ↔ AskUserQuestion
        ↓
Human Gate
        ↓
risk-analysis ↔ AskUserQuestion
        ↓
test-model-impact-analysis ↔ AskUserQuestion
        ↓
Candidate Discovery: test-model.md → «Тип проверки» → Zephyr keys
        ↓
Candidate Inspection: Jira/Zephyr read → actual test cases
        ↓
test-case-candidates.md/json + test-model-impact.md/json
        ↓
Human Gate
        ↓
test-design ↔ AskUserQuestion
        ↓
Human Gate
        ↓
checklist-writer
        ↓
checklist.md/json
        ↓
┌────────────────────────────────────────────────────┐
│                                                    │
↓                                                    ↓
test-case-writer                           test-case-maintainer
↓                                                    ↓
NEW only                                      EXISTING only
↓                                                    ↓
new-test-cases.md/json                  test-case-updates.md/json
(full new cases)                       (exact field/step patches)
└─────────────────────────┬──────────────────────────┘
                          ↓
             Human Gate: review all test changes
                          ↓
                test-model-maintainer
                          ↓
                     Human Gate
                          ↓
               pre-export-validation
                          ↓
       effective test set coverage validation
                          ↓
                 tms-export preparation
                          ↓
            READY_FOR_EXTERNAL_EXPORT
                          ↓
                external IDE TMS plugin
```

## Why two test artifacts

### `new-test-cases.md/json`

Только cases, которых раньше не существовало. Содержат полный test-case body по Project Test Case Profile. Export action = CREATE.

### `test-case-updates.md/json`

Только существующие cases, которые нужно изменить. Вместо полного rewrite указываются exact identity и изменения уровня поля/шага:

`field/step → before → after → reason → source refs`.

Export action = UPDATE. `before` используется как conflict guard.

## Effective coverage

Перед экспортом coverage проверяется по будущему состоянию:

`existing test corpus + approved update patches + approved new test cases`.

Это позволяет не считать update/new изолированно и не потерять test condition при split/переносе responsibility.

## Checklist

`checklist.md` — простой team-readable набор «что проверить» без detailed steps. Traceability остаётся в `checklist.json`. Checklist не является independent coverage.

## Project Test Case Contract

Writer/maintainer используют `qa-context/qa/test-cases/test-case-profile.yaml`. Если профиль не configured, flow ждёт настройки, а не угадывает поля/steps/expected.

## TMS handoff

`tms-export` не вызывает TMS и не перегенерирует tests. `export-plan.json` ссылается на два approved sources: CREATE bodies и UPDATE patches. Внешний IDE plugin выполняет операции и возвращает actual TMS keys/status.

## Runtime

```text
.agent-runs/<change-id>/
├── state.json
├── progress.md
├── context/
├── clarifications/
└── artifacts/
```

## Human Gates / clarifications

AskUserQuestion уточняет факты во время анализа. Human Gate отдельно утверждает stage. Модель не может self-approve.

## Full mode

После design/TMS handoff может выполняться implementation analysis, automation, spec-vs-code, executions, failure analysis, spec-vs-tests, coverage, final-conformance-check и qa-verdict.

## Existing test case discovery and inspection

`test-model-impact-analysis` выполняет две фазы.

### A. Candidate Discovery

1. Получает change signals из `context-discovery` и approved analysis.
2. В текущем `test-model.md` сначала ищет совпадения в поле **`Тип проверки`**.
3. Из найденных rows берёт exact existing Zephyr/TMS keys.
4. При наличии downstream effects расширяет candidates по связанным business/entity/state/API/topic terms в той же существующей матрице.
5. Создаёт `test-case-candidates.md/json`.

На этой фазе совпадение не означает `UPDATE`.

### B. Candidate Inspection

1. По exact keys читает actual current test cases через доступную Jira/Zephyr read integration по `TMS_READ_CONTRACT.md`.
2. Сопоставляет body existing TC с approved new behavior/risk/test obligations.
3. Только после этого ставит `KEEP / UPDATE / OBSOLETE / REVIEW`.
4. `ADD` допускается только после проверки primary + related existing candidates на semantic duplicate.

Если material candidate нельзя прочитать, stage не угадывает содержание и не передаёт maintainer фиктивный baseline.

`test-case-maintainer` не занимается поиском affected TC: он получает approved exact identity + current retrieved body и формирует только field/step-level patch plan.

`existing-test-cases-snapshot.json` сохраняет actual current body успешно прочитанных candidates. Это baseline для resume и `test-case-maintainer`; patch `before` должен строиться из snapshot, а не из повторного semantic search.
