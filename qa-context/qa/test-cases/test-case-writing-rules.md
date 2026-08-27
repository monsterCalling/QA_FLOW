# Manual Test Case Writing Rules

## 1. Главный принцип атомарности

**Один test case = одна основная test objective + один связный business scenario + однозначный основной outcome.**

Создавай отдельный test case, если меняется хотя бы один существенный аспект:

- business outcome;
- state transition;
- отдельная error branch;
- retry/duplicate/idempotency как самостоятельное поведение;
- concurrency/race condition;
- authorization condition, меняющая ожидаемый результат;
- существенная precondition;
- test data, превращающие проверку в другой сценарий.

Не создавай отдельный test case только потому, что один сценарий нужно подтвердить через API + DB + Kafka + logs. Это разные assertions одного business outcome.

## 2. Запрещённый mega-case

Плохо: один кейс последовательно проверяет happy path, validation error, timeout, retry, duplicate и recovery.

Такой case должен быть разбит на независимые сценарии, например:

- успешная обработка;
- business reject;
- timeout downstream;
- retry после timeout;
- duplicate event;
- recovery после restart.

## 3. Title

Title должен описывать действие/условие и ожидаемый business scenario.

Хорошо:

- `Отклонение заявки при ответе REJECTED от scoring`;
- `Повторная обработка одного eventId без повторного business side effect`.

Плохо:

- `Проверка заявки`;
- `Позитивный и негативный сценарий`;
- `Kafka test`.

## 4. Preconditions

Preconditions содержат только условия, которые должны существовать **до первого шага**. Не прячь setup в середине test steps.

## 5. Test data

Используй подтверждённые значения или явные placeholders. Не придумывай реальные идентификаторы, клиентов, счета, product codes, секреты и production data.

## 6. Steps

Один step = одно понятное business/technical action. Не объединяй несколько альтернативных путей в один step.

Количество шагов не является целью. Если кейс требует длинной цепочки из-за реального E2E flow — это допустимо, но writer обязан проверить, не начался ли внутри другой самостоятельный scenario.

## 7. Expected result

Модель expected result задаётся `test-case-profile.yaml`:

- `per_step` — expected result на каждый шаг;
- `final` — один итоговый expected result;
- `per_step_and_final` — оба уровня.

Writer не выбирает формат самостоятельно.

## 8. API / DB / Kafka / Logs

Добавляй только те verification points, которые доказывают expected behavior текущего case.

Не вставляй SQL, payload или log query «для полноты». Техническая проверка должна иметь конкретную цель.

## 9. Traceability

Каждый case должен ссылаться минимум на:

- approved test condition;
- requirement;
- risk — если case реализует risk obligation.

Новый существенный test condition нельзя молча изобрести внутри case. Верни gap в `test-design`.

## 10. Self-check атомарности

Перед выдачей каждого case writer отвечает:

1. Здесь одна primary objective?
2. Здесь один основной business outcome?
3. Нет ли внутри альтернативных success/error branches?
4. Retry/duplicate/concurrency не спрятаны внутри другого scenario?
5. Все technical assertions доказывают тот же outcome?

Если нет — `ATOMICITY_VIOLATION` и split до Human Review.


## Existing case update rules

Для existing test case не создавай полный переписанный дубль, если change локальный. Формируй точный patch:

- exact TMS/local identity;
- affected field или step;
- текущее `before`;
- требуемое `after`;
- причина;
- requirement/test-condition refs.

Для step-level update обязательно указывать номер шага и конкретную часть (`action`, `test_data`, `expected_result`).

Если существующий case должен быть разделён, TMS identity остаётся только у одной responsibility; остальные scenarios оформляются как новые cases.
