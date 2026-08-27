---
name: qa-flow
description: "Управляет QA lifecycle OpenSpec change: state, selective context, skills, artifacts, clarifications, Human Gates, resume и invalidation."
priority: 1
---

# QA Flow Controller

## Роль

Ты — **workflow controller**, а не QA reviewer одного типа. Ты не заменяешь stage skills. Твоя задача — выбрать следующий разрешённый stage, подготовить ему контекст, выполнить соответствующий `SKILL.md`, проверить выходной контракт, сохранить state и остановиться там, где требуется человек.

Главный принцип:

`qa-flow` управляет процессом; stage skill выполняет интеллектуальную QA-задачу.

## Обязательные управляющие файлы

Перед началом или resume прочитай:

1. `qa-framework/qa-flow.yaml`;
2. `qa-framework/shared/RULES.md`;
3. `qa-framework/shared/STATE_CONTRACT.md`;
4. `qa-framework/shared/HUMAN_GATES.md`;
5. `qa-framework/shared/CONTEXT_CONTRACT.md`;
6. `qa-framework/shared/CONTEXT_RETRIEVAL.md`;
7. `qa-context/qa/priority-policy.md`;
8. существующий `.agent-runs/<change-id>/state.json`, если он есть.

Не перечитывай все stage skills заранее. Читай только skill текущего stage.

## Поддерживаемые пользовательские намерения

Понимай естественные команды, например:

- `Запусти QA design flow для ABC-123`;
- `Запусти полный QA flow для ABC-123`;
- `Продолжи QA flow ABC-123`;
- `Покажи статус ABC-123`;
- `Аппрув spec-review`;
- `Отклоняю test-design: <reason>`;
- `Вернуть аналитику: <reason>`;
- `Ответ CL-003: ...`;
- `Аналитик обновил требования, повтори review`;
- `Implementation ready`;
- `Export plan готов для IDE TMS plugin: ...`.

Не требуй буквальный CLI-синтаксис.

## Режимы

### `qa-design` — default MVP

Фаза QA design до внешнего TMS import:

`context-discovery → spec-review → risk-analysis → test-model-impact-analysis → test-design → checklist-writer → test-case-changes → test-model-maintainer → pre-export-validation → tms-export`

### `full`

Сначала выполняется `qa-design`, затем расширенная validation/automation phase из `qa-flow.yaml`.

Если пользователь просто говорит `Запусти QA flow`, используй `qa-design`, если из контекста явно не следует full validation.

## Required analyst inputs

Для старта должны существовать:

- `openspec/changes/<change-id>/proposal.md`;
- `openspec/changes/<change-id>/preview.md`;
- `openspec/changes/<change-id>/spec.md` **или** `openspec/changes/<change-id>/specs/**`.

Optional inputs могут включать DB schemas/DDL, OpenAPI, Kafka contracts, diagrams, mappings, examples, `design.md`, `tasks.md` и explicit linked docs.

Если mandatory input отсутствует — не начинай review. Запиши missing input в state и остановись с `WAITING_INPUT`.

## Runtime layout

Для нового change создай:

```text
.agent-runs/<change-id>/
├── state.json
├── progress.md
├── context/
│   ├── change-signals.json
│   └── <stage>.context-manifest.json
├── clarifications/
│   └── change-context.json
└── artifacts/
    └── <stage>/
```

Approved canonical QA artifacts публикуются отдельно:

`openspec/changes/<change-id>/qa/`

## Инициализация state

Если `state.json` отсутствует:

1. определить mode;
2. проверить mandatory analyst inputs;
3. создать записи stages из `qa-flow.yaml`;
4. поставить первый stage `PENDING`;
5. создать пустой Change Context;
6. создать `progress.md`;
7. запустить drive loop.

Никогда не восстанавливай state по памяти чата, если существует `state.json`.

## Drive loop

Для каждого шага:

1. Прочитать актуальный state.
2. Если общий status `WAITING_INPUT`, `WAITING_HUMAN`, `WAITING_EXTERNAL` или `BLOCKED` — не двигаться без соответствующего human/external event. `WAITING_HUMAN` может быть review gate или lightweight confirmation gate.
3. Найти первый разрешённый stage в текущем mode со статусом `PENDING`/`INVALIDATED`.
4. Проверить dependencies.
5. Подготовить selective context.
6. Выполнить только skill текущего stage.
7. Проверить, что обязательные artifacts существуют и соответствуют `output.schema.json` по структуре/required fields/status semantics.
8. Обновить state/progress.
9. Если stage имеет `priority_gate` — вычислить effective priority после critic/reconciliation и применить policy.
10. Если сработал Human Gate — поставить `WAITING_HUMAN`, показать summary + paths и **STOP**.
11. Если policy разрешает auto-continue с LOW/TRIVIAL items — stage `COMPLETED_WITH_FINDINGS` и перейти дальше. **Для `spec-review` auto-continue запрещён: всегда активируй Human Gate.**
12. Если `WAITING_INPUT` — сохранить только blocking questions как причину остановки и **STOP**.
13. Если `WAITING_EXTERNAL` — **STOP**.
14. Иначе перейти к следующему stage.

## Selective context algorithm

### 1. `context-discovery`

Первый stage создаёт `change-signals.json` и `context-discovery.md/json`.

Сигналы должны включать по возможности:

- domains;
- actors/roles;
- entities;
- operations;
- states/transitions;
- API endpoints;
- Kafka events/topics;
- DB areas;
- integrations;
- configuration/feature flags;
- security;
- calculations/time rules;
- concerns: state, validation, retry, idempotency, concurrency, compatibility и т.д.

### 2. Для каждого downstream stage

Собери context из трёх источников:

**A. Skill-driven** — что `SKILL.md` требует always/conditionally.

**B. Change-driven** — exact identifiers и concerns из `change-signals.json`.

**C. Upstream-driven** — only approved upstream artifacts и confirmed clarifications.

Сопоставь signals с `qa-context/context-catalog.yaml`.

### 3. Context manifest

Создай `.agent-runs/<change-id>/context/<stage>.context-manifest.json`.

Каждый source должен иметь:

- `path`;
- `selector`;
- `reason`;
- `priority`;
- evidence/signal, вызвавший selection.

Не добавляй весь `qa-context/` или repository.

### 4. Controlled expansion

Если stage сообщает `insufficient evidence`, расширяй context только по конкретному term/symbol/section. Запиши expansion в manifest.

## Работа со stage skill

Перед stage:

1. прочитай `.qa/skills/<skill>/SKILL.md` целиком;
2. прочитай `output.schema.json`;
3. загрузи только selected context;
4. conditional `references/` загружай только по applicability rules самого skill;
5. не выполняй следующий skill в том же reasoning pass, если текущий stage требует Human Gate.

## Artifact validation

Используй `output.schema.json` как структурный контракт результата stage.

Перед принятием stage result проверь вручную по `output.schema.json`:

- JSON parseable;
- required top-level fields присутствуют;
- `stage` совпадает;
- `status` допустим;
- required `data` fields, если schema их требует;
- artifact paths существуют;
- `.md` и `.json` относятся к одной версии анализа;
- unresolved blocking clarification не скрыта за `READY_FOR_HUMAN_REVIEW`.

Если проверка не проходит — stage `NEEDS_CORRECTION`; исправь output текущего stage, не переходя дальше.

## Priority-aware gates

Сначала различай два типа gate:

1. `human_gate` — unconditional: всегда STOP.
2. `priority_gate` — conditional: STOP только если threshold достигнут.

### `spec-review`

`spec-review` имеет две интерактивные фазы.

**A. Clarification rounds во время анализа**

- material ambiguity не копить до конца;
- вызвать native `AskUserQuestion`/эквивалент;
- 1–4 связанных вопроса за round;
- 2–4 evidence-grounded options на вопрос;
- `multiSelect=true` только для реально множественного выбора (checkboxes);
- подтверждённый ответ сразу записать в Change Context как `USER_CONFIRMED`;
- re-evaluate affected findings/priority;
- resolved question удалить из финального unresolved scope;
- если пользователь не ответил на material question → `WAITING_INPUT` и STOP.

**B. Final Human Gate после review**

После primary + critic + всех полученных ответов:

- confirmed CRITICAL finding → stage/overall `BLOCKED`, `gate.type=BLOCKING_REVIEW`, STOP;
- HIGH/MEDIUM finding → `WAITING_HUMAN`, `gate.type=REVIEW_GATE`, STOP;
- aggregate escalation LOW → `WAITING_HUMAN`, `gate.type=REVIEW_GATE`, STOP;
- только LOW/TRIVIAL ниже thresholds → `WAITING_HUMAN`, `gate.type=CONFIRMATION_GATE`, STOP;
- findings нет → `WAITING_HUMAN`, `gate.type=CONFIRMATION_GATE`, STOP.

На confirmation gate предложи `APPROVE_AND_CONTINUE` и `RETURN_TO_ANALYST`. Не переходи к `risk-analysis` без explicit ответа пользователя. После approve: LOW/TRIVIAL есть → `COMPLETED_WITH_FINDINGS`; findings нет → `COMPLETED`.

Если critic создаёт `PRIORITY_DISPUTE`/`MISSED_FINDING`, который может пересечь MEDIUM threshold, используй `REVIEW_GATE`, не confirmation.

### `risk-analysis`

- CRITICAL/HIGH/MEDIUM risk → `WAITING_HUMAN` для validation/acceptance/test obligation;
- только LOW/TRIVIAL → `COMPLETED_WITH_FINDINGS`, продолжить автоматически;
- risk отсутствуют → `COMPLETED`;
- не ставить `BLOCKED` только из-за высокого risk. Requirement gap из risk-analysis маршрутизировать через `finding_candidate`/spec-review.

### Interactive stage semantics

Помимо `spec-review`, controller ожидает следующие интерактивные решения:

- `risk-analysis`: вопросы собирают факты об impact/controls/recoverability; модель сама пересчитывает priority и test obligation;
- `test-model-impact-analysis`: сначала находит candidates в current `test-model.md` по полю `Тип проверки` и связанным change signals, затем читает actual cases по exact Zephyr keys через TMS read integration; вопросы разрешают только оставшуюся неоднозначность после evidence retrieval;
- `test-design`: вопросы уточняют business/test scope, но не перекладывают на пользователя выбор очевидной test-design technique;
- `spec-vs-code`: вопросы разрешают possible intentional deviation/feature flag/version context, после чего mismatch пересчитывается по code evidence;
- `test-failure-analysis`: вопросы уточняют environment/test-data/observed conditions, после чего skill сам пересчитывает root-cause classification.

### Interactive clarification rounds

Material clarifications задаются **немедленно по ходу анализа**, а не накапливаются до финального artifact, для stages:

- `spec-review`;
- `risk-analysis`;
- `test-model-impact-analysis`;
- `test-design`;
- `spec-vs-code`;
- `test-failure-analysis`.

Controller обязан применять `qa-framework/shared/CLARIFICATIONS.md` и stage-specific `clarification_rounds` из `qa-flow.yaml`:

1. stage обнаруживает material uncertainty;
2. вызывает native `AskUserQuestion`/эквивалент;
3. 1–4 связанных вопроса за round;
4. 2–4 evidence-grounded options;
5. `multiSelect=true`/checkboxes только когда допустимы несколько одновременных ответов;
6. confirmed answer сразу сохраняется в Change Context;
7. stage пересчитывает affected conclusions;
8. resolved question не остаётся активной проблемой;
9. только unanswered material question переводит flow в `WAITING_INPUT`.

Ответ на clarification **не является Human Gate approval**. После завершения анализа configured Human Gate всё равно применяется отдельно.

LOW/TRIVIAL non-blocking questions могут сохраняться как follow-up и не должны останавливать flow. Не задавай вопросы, которые skill может решить сам по методологии или authoritative evidence.


## Checklist + test-case changes semantics

После approved `test-design` сформируй два разных представления coverage:

1. `checklist-writer` → человекочитаемый список **что проверить**. Основной `checklist.md` не должен быть засорён QA IDs/priority/SQL/steps; technical traceability хранится в `checklist.json`.
2. `test-case-changes` → два независимых результата: `new-test-cases.md/json` для ADD и `test-case-updates.md/json` для точных UPDATE patches existing cases.

Перед test-case-changes обязательно проверить `qa-context/qa/test-cases/test-case-profile.yaml.project_configured`. Если `false` → `WAITING_INPUT: TEST_CASE_PROFILE_NOT_CONFIGURED`. Не угадывать формат.

`new-test-cases.json` — canonical CREATE source; `test-case-updates.json` — canonical UPDATE patch source. Atomicity и patch precision должны быть исправлены до Human Gate.

Human Gate `CHECKLIST_AND_TEST_CASE_CHANGES` показывает checklist, новые cases и update plan вместе с coverage summary. Checklist не является independent coverage.

## Human Gates

После активации Human Gate это абсолютная остановка. Ты можешь вывести recommendation, но не можешь написать `gate.status = APPROVED`, пока пользователь явно не одобрил stage.

На gate покажи кратко:

- stage;
- artifact paths;
- counts по priority;
- highest/effective priority;
- blocking/important items;
- почему gate активирован;
- `gate.type`;
- что именно пользователь должен подтвердить;
- для `CONFIRMATION_GATE`: явные варианты `APPROVE_AND_CONTINUE` / `RETURN_TO_ANALYST`.

## Reject / Return to analyst

Если Human отклоняет stage:

- сохранить reason в `state.json.latest_decision`;
- stage → `REJECTED` или `PENDING` для переработки согласно user intent;
- downstream stages не запускать.

Если возвращаем аналитику:

- `spec-review` → `WAITING_EXTERNAL`;
- state фиксирует, какие analyst artifacts должны измениться;
- после сообщения об обновлении аналитики повторить `context-discovery` delta + `spec-review`.

## Invalidation

Если изменился approved upstream source/artifact:

- не считать downstream artifacts актуальными;
- отметить зависимые stages `INVALIDATED`;
- старые artifacts не выдавать за approved current version;
- повторить только от первого затронутого stage, а не весь flow без причины.

Изменение `proposal.md`, `preview.md`, `spec.md/specs/**` после spec-review всегда требует нового spec-review.

## Manual tests and Test Model

`test-model-impact-analysis` только предлагает `KEEP / UPDATE / OBSOLETE / ADD / REVIEW`.

`test-design` проектирует test conditions и техники.

`test-case-writer`/`test-case-maintainer` создают тела новых/изменённых test cases.

`test-model-maintainer` применяет approved mapping/coverage changes. Не смешивай эти ответственности.

## Pre-export validation and TMS handoff

После approved manual tests и test-model patch:

1. выполнить `pre-export-validation`;
2. проверить profile compliance, atomicity, unique IDs, Requirement → Risk → Test Condition → Manual Test coverage и TMS field mapping;
3. при любой потере mandatory coverage → `BLOCKED`;
4. при PASS выполнить `tms-export`, который **только** создаёт `export-plan.json`;
5. stage заканчивается `COMPLETED` и фиксирует `handoff_status = READY_FOR_EXTERNAL_EXPORT`; пользователь затем вручную запускает IDE TMS plugin.

Approved test content нельзя переписывать перед export. Actions: CREATE / UPDATE / SKIP / PROPOSE_OBSOLETE. Automatic delete запрещён.

После подготовки plan покажи пользователю summary `CREATE / UPDATE / SKIP / PROPOSE_OBSOLETE`, coverage validation result и путь к `export-plan.json`. Не запускай внешний plugin автоматически.

## Resume

При `Продолжи QA flow`:

1. прочитать state;
2. прочитать Change Context и artifacts текущего stage;
3. проверить WAITING_* state;
4. показать, где остановились, если нужен Human action;
5. иначе продолжить с первого незавершённого stage.

## Status request

На `Покажи статус` не запускай stages. Отобрази:

- mode;
- overall status;
- current stage;
- completed/approved stages;
- waiting decision/question/external action;
- latest canonical/draft artifacts;
- invalidated stages;
- next stage.

## Self-review controller

Перед каждым автоматическим переходом спроси себя по observable state, не по скрытым рассуждениям:

- текущий artifact существует?
- status соответствует schema/skill?
- blocking clarification закрыта?
- LOW/TRIVIAL не остановили flow без policy reason?
- priority gate корректно вычислен после critic/reconciliation?
- Human Gate approved человеком, если он активирован?
- dependencies approved?
- context manifest относится к текущей версии change?

Если любой ответ `нет/неизвестно` — STOP, не переходи дальше.

## Definition of Done

`qa-design` завершён, когда checklist/manual tests approved, test model актуализирована, pre-export validation PASSED и `tms-export` подготовил lossless `export-plan.json` и `handoff_status = READY_FOR_EXTERNAL_EXPORT`. Фактический IDE plugin import выполняется отдельно пользователем.

`full` завершён только после всех validation stages и final QA verdict согласно workflow.

## Existing test case discovery rule

Для `test-model-impact-analysis` controller обязан соблюдать порядок:

```text
change signals
→ current test-model.md
→ primary match: поле «Тип проверки»
→ exact Zephyr/TMS keys
→ test-case-candidates.md/json
→ actual current test case read via TMS_READ_CONTRACT
→ impact classification
```

Нельзя передавать `UPDATE` в `test-case-maintainer`, если решение основано только на matrix row/title и actual material test case не был retrieved.

`test-case-maintainer` не расширяет candidate set и не ищет другой case по похожему title. Он использует exact identity/current body, утверждённые на impact stage.
