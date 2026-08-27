# Automation Rules

## 1. Назначение

Документ задаёт правила выбора сценариев для автоматизации и требования к автотестам.

## 2. Что автоматизировать в первую очередь

Высокий приоритет:

- P0/P1 regression scenarios;
- критичные business flows;
- стабильные REST API;
- deterministic Kafka flows;
- state transitions;
- расчёты;
- валидацию данных;
- repeatable integration scenarios;
- проверки, выполняемые в каждом релизе;
- сценарии с высокой стоимостью ручного прогона.

## 3. Что не следует автоматизировать автоматически

Низкий приоритет или manual-first:

- одноразовые migration checks;
- экспериментальная функциональность;
- визуальная оценка без подходящих инструментов;
- сценарии с нестабильной неконтролируемой зависимостью;
- редко выполняемые проверки с высокой стоимостью поддержки;
- сценарии, где автоматизация не даёт воспроизводимого результата.

## 4. Уровень автоматизации

Предпочитать наиболее низкий уровень, который надёжно доказывает требование.

Возможные уровни:

- unit;
- component;
- integration;
- API;
- Kafka integration;
- DB integration;
- E2E.

Не использовать E2E, если требование можно надёжно проверить на component/integration/API уровне.

## 5. Требования к автотесту

Автотест должен:

- иметь понятную связь с requirement/test condition;
- проверять business outcome;
- иметь детерминированные assertions;
- быть воспроизводимым;
- не зависеть от порядка запуска;
- не зависеть от случайных существующих данных;
- корректно подготавливать данные;
- выполнять cleanup;
- использовать existing helpers/fixtures;
- соответствовать стилю test framework проекта;
- иметь понятное имя;
- иметь диагностически полезное падение.

## 6. Запрещённые практики

Без обоснования запрещены:

- `sleep` вместо ожидания условия;
- assertions только на HTTP 200;
- catch exception и игнорирование;
- зависимости между тестами;
- hardcoded shared test data;
- неограниченные retries;
- flaky timeout;
- mock всего приложения;
- проверка внутренних implementation details вместо поведения.

## 7. Async / Kafka

Для async тестов:

- использовать polling/await condition;
- иметь верхний timeout;
- проверять отсутствие duplicate side effects;
- учитывать eventual consistency;
- очищать/изолировать test messages;
- использовать уникальные correlation/business IDs.

## 8. БД

DB assertions допустимы, если БД является значимой частью observable behavior или необходима для подтверждения business state.

Не следует проверять каждую внутреннюю колонку, если она не относится к требованию.

## 9. External Systems

Предпочтение:

1. controlled test dependency;
2. project stub/mock infrastructure;
3. sandbox;
4. реальная внешняя система — только когда это необходимо.

## 10. Review

Перед merge сгенерированные LLM автотесты должны проходить:

1. independent LLM review;
2. Human QA review.

Reviewer проверяет:

- соответствие requirement;
- достаточность assertions;
- отсутствие false-positive test;
- стабильность;
- data isolation;
- cleanup;
- поддерживаемость.
