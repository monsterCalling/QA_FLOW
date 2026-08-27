# Test Model Reference

## Назначение

В этом проекте `test-model.md` — матрица покрытия с ключами существующих Jira Zephyr test cases.

Этот baseline-файл не должен становиться второй копией реальной test model.

## Подключение реальной test model

Если реальный файл находится, например, в `docs/qa/test-model.md`, зарегистрируйте его в `qa-context/context-catalog.yaml` как QA source.

Пример:

```yaml
- id: project-test-model
  type: qa
  file: docs/qa/test-model.md
  description: Existing project test coverage model
  tags:
    - test-model
    - coverage
    - regression
    - zephyr
```

`context-discovery` и `qa-flow` controller будут включать только релевантные разделы test model в Context Manifest текущего stage.

Если test model хранится непосредственно в `qa-context/qa/test-model.md`, замените этот reference реальным файлом.

## Правила работы

Агент обязан:

- использовать актуальную test model как source of truth;
- сохранять существующие Zephyr keys;
- не создавать дубли;
- классифицировать влияние change как `KEEP / UPDATE / OBSOLETE / ADD / REVIEW`;
- получить Human Approve до изменения матрицы;
- получить Human Approve фактического diff после изменения.

Для нового теста без Zephyr key используется локальный ID и статус:

```text
MT-001
PENDING_ZEPHYR_KEY
```

Тела новых/изменённых ручных тестов хранятся отдельно:

- `new-test-cases.md/json`;
- `test-case-updates.md/json`.
