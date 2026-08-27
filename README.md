# QA FLOW FOR SDD v.1.0

QA framework для SDD/OpenSpec в GigaCode/LLM runtime. Управление — meta-skill `.qa/skills/qa-flow/SKILL.md`, file-based state, selective context и Human Gates.

## Быстрый старт

Расположите framework в корне проекта рядом с `openspec/`.

```text
openspec/changes/<change-id>/
├── proposal.md
├── preview.md
└── spec.md        # или specs/**
```

Запуск:

```text
Запусти QA design flow для CARD-1234.
Используй .qa/skills/qa-flow/SKILL.md как управляющий skill.
```

## Default QA Design Flow

```text
context-discovery
→ spec-review + Human Gate
→ risk-analysis
→ test-model-impact-analysis + Human Gate
→ test-design + Human Gate
→ checklist-writer
→ test-case-writer: new-test-cases.md/json
→ test-case-maintainer: test-case-updates.md/json
→ Human Gate: checklist + new cases + update plan
→ test-model-maintainer + Human Gate
→ pre-export-validation
→ tms-export preparation
→ READY_FOR_EXTERNAL_EXPORT
→ пользователь запускает IDE TMS plugin
```

## Checklist: для команды

`checklist.md` — короткий человекочитаемый набор **что проверить**, без SQL, detailed steps и служебных IDs в основном тексте. `checklist.json` хранит traceability.

## Manual test changes: теперь два отдельных результата

### 1. Новые тест-кейсы

```text
new-test-cases.md
new-test-cases.json
```

Содержат **только новые test cases** полностью: поля, preconditions, test data, steps, expected results и traceability по Project Test Case Profile.

### 2. Изменения существующих тест-кейсов

```text
test-case-updates.md
test-case-updates.json
```

Для каждого existing case указываются:

- exact Zephyr/TMS key / local ID;
- причина update;
- какие поля затронуты;
- какой конкретно step меняется;
- `before`;
- `after`;
- requirement / test-condition refs.

Не нужно читать полностью переписанный case, чтобы понять change.

## Atomicity

**Один test case = одна primary objective + один business scenario + один основной outcome.** Happy path, reject, timeout, retry, duplicate и recovery не объединяются в mega-case.

## Project profile

```text
qa-context/qa/test-cases/
├── test-case-profile.yaml
├── test-case-writing-rules.md
├── test-case-examples.md
└── tms-field-mapping.yaml
```

Пока `project_configured: false`, writer/maintainer останавливаются с `TEST_CASE_PROFILE_NOT_CONFIGURED` вместо угадывания формата.

## Pre-export / coverage

До TMS строится **effective test set**:

```text
existing cases
+ approved test-case-updates patches
+ approved new-test-cases
```

После этого проверяется `Requirement → Risk → Test Condition → Effective Manual Test`. Потеря mandatory coverage блокирует export.

## TMS / IDE plugin

```text
new-test-cases.json ──────┐
                          ├→ pre-export-validation → export-plan.json → IDE plugin → Zephyr
 test-case-updates.json ──┘
```

CREATE берётся только из `new-test-cases.json`. UPDATE — только из `test-case-updates.json`.

Для UPDATE `before` является guard: если TMS case уже изменился и current value не совпадает, plugin должен вернуть conflict, а не перезаписать его вслепую.

## Runtime state

```text
.agent-runs/<change-id>/
├── state.json
├── progress.md
├── context/
├── clarifications/
└── artifacts/
```

Approved artifacts публикуются в `openspec/changes/<change-id>/qa/`.

## Как теперь определяются existing test cases для UPDATE

`test-model.md` используется как индекс, а не как источник полного test case body.

```text
change signals
    ↓
test-model.md
    ↓
поиск по полю «Тип проверки»
    ↓
Zephyr/TMS keys кандидатов
    ↓
test-case-candidates.md/json
    ↓
Jira/Zephyr read integration
    ↓
actual current test cases
    ↓
existing-test-cases-snapshot.json
    ↓
KEEP / UPDATE / OBSOLETE / REVIEW
```

Если change имеет downstream effect, candidate discovery дополнительно проверяет связанные строки текущей test model по business/entity/state/API/topic terms, чтобы не потерять косвенно затронутое coverage.

Совпадение `Тип проверки = Kafka` само по себе **не означает UPDATE**. Решение принимается только после чтения фактического test case по exact Zephyr/TMS key. Если material existing case нельзя получить, flow не угадывает его содержимое.

Создаются два impact artifacts:

- `test-case-candidates.md/json` — какие existing cases были отобраны для inspection и почему;
- `test-model-impact.md/json` — итоговая классификация после чтения actual cases.

`test-case-maintainer` больше не ищет affected cases самостоятельно: он получает approved exact keys + current bodies и формирует только точный `before → after` patch plan.
