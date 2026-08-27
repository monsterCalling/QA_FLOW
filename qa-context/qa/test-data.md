# Test Data Rules

## 1. Назначение

Документ задаёт универсальные правила подготовки и использования тестовых данных.

## 2. Основные принципы

Тестовые данные должны быть:

- контролируемыми;
- воспроизводимыми;
- минимальными;
- понятными;
- изолированными;
- очищаемыми;
- не содержать реальные чувствительные данные.

## 3. Data Sets

Для test condition рассмотреть:

- valid;
- invalid;
- boundary;
- empty;
- null;
- duplicate;
- expired;
- inactive;
- terminal state;
- conflicting state;
- historical/versioned.

## 4. Уникальность

Для параллельного выполнения использовать уникальные:

- IDs;
- correlation IDs;
- event IDs;
- business keys;
- usernames, если применимо;
- external references.

Не использовать один shared entity для независимых автотестов.

## 5. Preconditions

Каждый тест должен явно понимать:

- какие сущности должны существовать;
- их статус;
- обязательные связанные записи;
- настройки;
- feature flags;
- состояние внешних систем;
- необходимые Kafka offsets/events.

## 6. Подготовка данных

Приоритет способов:

1. публичный/служебный API;
2. специальные test helpers;
3. fixtures;
4. controlled DB setup;
5. ручное создание.

Прямое изменение БД использовать только если:

- это допускается проектом;
- невозможно корректно подготовить состояние через API;
- данные после подготовки остаются согласованными.

## 7. Cleanup

После теста:

- удалить созданные данные, если это допустимо;
- либо перевести их в безопасное terminal state;
- очистить тестовые Kafka messages, если инфраструктура это позволяет;
- не ломать данные других тестов;
- не удалять shared fixtures.

## 8. Sensitive Data

Запрещено использовать:

- реальные персональные данные;
- реальные банковские реквизиты;
- production tokens;
- реальные secrets.

Использовать synthetic/masked data.

## 9. Time-Dependent Data

Для дат/времени учитывать:

- timezone;
- начало/конец дня;
- month/year boundary;
- leap year;
- expiration;
- future/past dates;
- clock skew, если применимо.

## 10. Failure Classification

Если тест упал из-за:

- отсутствующей precondition;
- некорректного fixture;
- stale data;
- collision;
- недоступного required entity;

классифицировать как `TEST_DATA`, если нет признаков product defect.
