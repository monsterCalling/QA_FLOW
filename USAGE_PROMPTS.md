# Usage Prompts

## Запуск

```text
Запусти QA design flow для <CHANGE-ID>.
Используй .qa/skills/qa-flow/SKILL.md как controller и selective context.
```

## Resume

```text
Продолжи QA flow <CHANGE-ID>. Сначала прочитай .agent-runs/<CHANGE-ID>/state.json и Change Context.
```

## Настройка Project Test Case Contract

```text
Проверь qa-context/qa/test-cases/test-case-profile.yaml и tms-field-mapping.yaml.
Покажи, какие project-specific поля ещё нужно заполнить перед test-case-changes/TMS export.
```

## После Human Approve test-case changes

```text
Продолжай flow: выполни pre-export-validation. Не изменяй approved new-test-cases.json и test-case-updates.json.
```

## Подготовка к IDE plugin

```text
Подготовь export-plan для внешнего IDE TMS plugin. Ничего не выгружай сам и не переписывай test cases.
```
