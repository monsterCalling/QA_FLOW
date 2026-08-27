# Approved Project Test Case Examples

> Замените примеры ниже на 5–10 реально утверждённых тест-кейсов вашей команды. Пока проектные примеры не добавлены, этот файл показывает только структуру и **не является источником business rules**.

## Example 1 — atomic business reject

**ID:** MT-001  
**Title:** Отклонение заявки при ответе REJECTED от внешней системы

**Objective:** Проверить один сценарий отклонения заявки.

**Preconditions**
- Существует тестовая заявка в исходном статусе `<SOURCE_STATUS>`.
- Stub внешней системы настроен на ответ `REJECTED`.

**Test data**
- applicationId: `<TEST_APPLICATION_ID>`

**Steps**
1. Отправить заявку на обработку.  
   **Expected:** запрос принят согласно контракту.
2. Дождаться обработки ответа `REJECTED`.  
   **Expected:** заявка получает подтверждённый спецификацией статус отказа.

**Verification**
- Persisted state соответствует expected business status.
- Если по спецификации публикуется событие, опубликовано корректное событие ровно в требуемом количестве.

**Traceability:** `<REQ-ID>`, `<TCND-ID>`

## Example 2 — duplicate as separate scenario

**ID:** MT-002  
**Title:** Повторная обработка одного eventId без повторного business side effect

**Objective:** Проверить идемпотентную/duplicate-обработку только если такое поведение подтверждено требованиями.

**Preconditions**
- Подготовлен event с фиксированным `<EVENT_ID>`.

**Steps**
1. Отправить event первый раз.  
   **Expected:** событие обработано согласно specification.
2. Повторно отправить тот же event.  
   **Expected:** повторная обработка соответствует подтверждённому duplicate/idempotency contract.

**Traceability:** `<REQ-ID>`, `<RISK-ID>`, `<TCND-ID>`
