# Regression Rules

## 1. Назначение

Документ задаёт базовые правила определения regression scope.

Regression scope формируется на основании:

- изменённого поведения;
- затронутого кода;
- зависимостей;
- test model;
- данных;
- интеграций;
- shared components.

## 2. Общий принцип

Нужно тестировать не только change, но и существующее поведение, которое может быть затронуто.

Каждый regression item должен иметь причину включения.

Запрещено автоматически включать «полный регресс» без анализа impact.

## 3. API

Если изменён существующий API, проверить:

- существующий happy path;
- старые обязательные/опциональные поля;
- старые consumers;
- validation;
- error contract;
- auth;
- backward compatibility.

Если изменена shared DTO/model:

- найти все endpoints, которые её используют;
- определить consumers;
- проверить serialization/deserialization.

## 4. Kafka

Если изменён Kafka event contract, проверить:

- producer;
- все consumers;
- schema compatibility;
- optional/required поля;
- enum;
- key;
- headers;
- old/new message compatibility;
- retry и idempotency.

Если изменена consumer logic:

- valid event;
- duplicate;
- retry;
- downstream failure;
- restart;
- ordering, если применимо.

## 5. Database

Если изменена schema, проверить:

- CRUD затронутой сущности;
- миграцию;
- существующие записи;
- default values;
- nullable;
- indexes/constraints;
- связанные запросы;
- history;
- batch jobs;
- отчёты/выборки.

Если изменена shared table:

- определить все сервисы/процессы, которые её читают или пишут.

## 6. State Machine

Если изменён статус или переход, проверить:

- весь изменённый переход;
- соседние переходы;
- запрещённые переходы;
- terminal states;
- повторный переход;
- history;
- интеграционные события;
- API/UI ограничения.

## 7. Расчёты

Если изменена формула или расчёт, проверить:

- все типы сущностей/продуктов, использующие расчёт;
- min/max;
- rounding;
- precision;
- null/zero;
- исторические сценарии;
- downstream consumers результата.

## 8. Shared Component / Library

Если изменена общая библиотека:

- найти все consumers;
- определить критичные способы использования;
- проверить backward compatibility;
- выполнить targeted regression для каждого типа использования.

## 9. Authorization

Если изменены роли/permissions, проверить:

- разрешённые роли;
- запрещённые роли;
- старые роли;
- direct object access;
- privilege escalation;
- UI visibility, если есть UI.

## 10. Integration

Если изменён adapter/mapper, проверить:

- mapping;
- missing fields;
- unknown fields;
- errors;
- timeout;
- retries;
- old contract;
- new contract;
- duplicate calls.

## 11. Определение глубины регрессии

### Minimal regression

Допустим, если:

- change локальный;
- нет shared components;
- нет contract change;
- impact подтверждён;
- есть хорошее existing coverage.

### Extended regression

Требуется, если:

- меняется shared logic;
- затронута state machine;
- изменяется contract;
- затронута БД;
- есть async processing;
- нет достаточного покрытия.

### Full regression

Используется только при обоснованной необходимости:

- крупный architectural change;
- major platform upgrade;
- migration;
- release с большим числом пересекающихся изменений;
- невозможно надёжно определить impact.
