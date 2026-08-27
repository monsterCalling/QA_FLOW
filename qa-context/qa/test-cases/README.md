# Project Test Case Contract

Эта папка задаёт **проектный стандарт ручного тест-кейса**. `test-case-writer` и `test-case-maintainer` обязаны использовать эти файлы и не должны угадывать формат по общему опыту модели.

Файлы:

- `test-case-profile.yaml` — обязательные/необязательные поля, модель expected result, правила test data и granularity;
- `test-case-writing-rules.md` — человекочитаемые правила атомарности и стиля;
- `test-case-examples.md` — 5–10 утверждённых примеров хороших тест-кейсов проекта;
- `tms-field-mapping.yaml` — mapping `new-test-cases.json` and `test-case-updates.json` → поля IDE plugin/TMS.

Перед первым использованием на проекте заполните профиль и примеры. Пока `project_configured: false`, manual test generation должна остановиться с `TEST_CASE_PROFILE_NOT_CONFIGURED`.

Важно: examples показывают стиль, но не являются источником business rules для текущего change.
