# Test Environment Rules

## 1. Назначение

Этот документ задаёт универсальные правила работы с тестовыми окружениями.

Конкретные названия стендов, URL, namespace и доступы должны быть добавлены проектной командой.

## 2. Environment Context

Для каждого стенда желательно фиксировать:

- назначение;
- URL;
- namespace;
- version/build;
- dependencies;
- DB;
- Kafka;
- stubs;
- feature flags;
- ограничения;
- способ доступа к логам.

## 3. Перед тестированием

Проверить:

- развернута ожидаемая версия;
- миграции применены;
- required services healthy;
- Kafka topics доступны;
- test dependencies доступны;
- feature flags корректны;
- test data подготовлены.

## 4. Environment vs Product Defect

Не классифицировать падение как product defect без проверки:

- health зависимостей;
- deploy status;
- конфигурации;
- сети;
- credentials;
- test data;
- stub state;
- Kafka availability.

## 5. Stubs

Для static/dynamic stubs фиксировать:

- какой внешний сценарий имитируется;
- request matcher;
- response;
- delay;
- error mode;
- call count;
- cleanup/reset.

Не считать тест E2E, если критичная внешняя интеграция заменена stub, если это важно для отчётности.

## 6. Configuration

При изменении конфигурации проверить:

- default value;
- environment-specific override;
- отсутствующую переменную;
- invalid value;
- restart/reload behaviour;
- secret availability.

## 7. Observability

Для расследования должны быть доступны:

- application logs;
- correlation IDs;
- deployment events;
- pod/container status;
- Kafka diagnostics;
- DB diagnostics, если разрешено.

## 8. Environment Issues

Примеры `ENVIRONMENT`:

- pod crash из-за некорректного стендового secret;
- недоступна общая тестовая зависимость;
- сломан stub;
- Kafka broker недоступен;
- не применена migration;
- развернута неверная версия;
- закончились ресурсы стенда.

## 9. Ограничения

Все известные ограничения стенда должны быть зафиксированы ниже командой проекта.

### Project-specific limitations

<!-- Заполнить для конкретного проекта -->
