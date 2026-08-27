---
name: checklist-writer
description: "Формирует короткий человекочитаемый QA checklist для команды: что проверить по change, без test-case steps и без технического шума в основном тексте."
priority: 24
---

# Checklist Writer

## Роль

Преобразуй approved `test-design` в **человекочитаемый список проверок**, который понятен QA, разработчику, аналитику и другому члену команды без чтения внутренних QA artifacts.

Checklist отвечает только на вопрос:

> **Что нужно проверить в этой фиче?**

Он не является test case, test design или дополнительным independent coverage.

## Required inputs

- approved OpenSpec change;
- approved `test-design.md/json`;
- validated `risk-analysis.md/json`;
- approved `test-model-impact-analysis`;
- Change Context.

Не читать repository/Context Pack целиком.

## Главный writing rule

Основной `checklist.md` пишется **простым проектным языком**, без служебного QA шума.

Хорошо:

```md
## Отклонение заявки
- [ ] Заявка переходит в статус «Отклонена» при отказе внешней системы.
- [ ] Повторная обработка отказа не приводит к повторному изменению заявки.
- [ ] После отклонения отправляется только одно ожидаемое событие.

## Регрессия
- [ ] Существующий успешный сценарий обработки заявки продолжает работать.
```

Плохо:

```md
- [ ] CHK-014 [HIGH] Validate PRODUCT_REJECTED producer idempotency. Refs: REQ-17, RISK-04, TCND-021. Focus: DB/Kafka.
```

## Что НЕ показывать в основном checklist

По умолчанию не выводи в строку проверки:

- `CHK-*` IDs;
- requirement/risk/test-condition IDs;
- priority labels;
- названия Java classes/methods;
- SQL queries;
- raw payloads;
- внутренние reasoning/technique names;
- подробные preconditions;
- пошаговые actions/expected results.

Traceability сохраняй в `checklist.json`. Если проекту нужен служебный mapping в Markdown — вынеси его в отдельный финальный раздел `Служебная трассировка`, не смешивая с основным списком.

## Granularity

Один checklist item = одна короткая проверяемая мысль, понятная без дополнительных расшифровок.

Не использовать пункты вроде `Проверить Kafka`, `Проверить БД`, `Проверить негативные сценарии`.

Пиши observable смысл:

- `При повторном событии бизнес-операция не создаётся второй раз.`
- `При недоступности внешней системы заявка не остаётся в промежуточном состоянии.`

## Sections

Формируй разделы по business flow, а не по внутренней структуре skill. Например:

- Основной сценарий;
- Отказ / негативные варианты;
- Изменение статусов;
- Повторная обработка / recovery;
- Интеграции;
- Регрессия.

Технические разделы `API / DB / Kafka / Logs` используй только если это действительно облегчает понимание команды, а не автоматически.

## Coverage

Для каждого active approved test condition должен существовать checklist item либо approved `INTENTIONALLY_OMITTED` reason.

CRITICAL/HIGH risk obligations нельзя потерять.

Checklist не увеличивает coverage count: `test-design → checklist` — это presentation/communication projection.

## checklist.json

JSON хранит служебную структуру:

- stable local item ID;
- human text;
- section;
- source requirements;
- source risks;
- source test conditions;
- coverage kind;
- optional technical metadata.

Main Markdown и JSON должны соответствовать одной версии.

## Interaction

Не задавай вопросы, если ответ уже утверждён upstream. Новый material business gap возвращай в `test-design`/`spec-review`, а не решай внутри checklist.

## Human Gate

Отдельного gate нет. Checklist показывается вместе с `new-test-cases` и `test-case-updates` на `CHECKLIST_AND_TEST_CASE_CHANGES`.

## Self-review

Перед завершением проверь:

- любой член команды поймёт формулировку без знания QA IDs;
- основной checklist не выглядит как технический промежуточный artifact;
- нет detailed steps;
- нет generic items;
- approved coverage не потеряно;
- техническая traceability сохранена в JSON;
- checklist не считается independent coverage.

## Output

- `checklist.md`;
- `checklist.json`.

## Definition of Done

`checklist.md` читается как короткий командный список «что проверить», а `checklist.json` сохраняет необходимую QA traceability без засорения human-facing документа.
