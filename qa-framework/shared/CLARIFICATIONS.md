# Clarification Contract

## 1. Цель

Не накапливать material вопросы до конца аналитического stage. Существенные неопределённости закрываются интерактивно по мере анализа, чтобы финальный artifact содержал минимальный остаточный scope неизвестностей.

Интерактивные clarification rounds обязательны для:

- `spec-review`;
- `risk-analysis`;
- `test-model-impact-analysis`;
- `test-design`;
- `spec-vs-code`;
- `test-failure-analysis`.

Остальные analytical skills могут использовать тот же протокол только при material uncertainty. Writer/maintainer skills задают вопрос лишь когда без ответа нельзя корректно выполнить уже approved решение.

## 2. Когда задавать вопрос сразу

Если ответ способен изменить expected behavior, contract/state/data semantics, priority, risk test obligation, test-model classification, test scope, mismatch classification, failure classification или readiness decision:

1. не финализируй затронутый вывод;
2. сначала проверь Change Context и authoritative source;
3. сформируй clarification round;
4. вызови native `AskUserQuestion`/эквивалент runtime;
5. после ответа пересчитай затронутую часть анализа;
6. продолжи stage.

Не спрашивай то, что уже подтверждено authoritative source/Change Context или что skill обязан определить методологически сам.

## 3. Формат AskUserQuestion

За один round — **1–4 связанных вопроса**. Каждый вопрос содержит:

```json
{
  "id": "CL-001",
  "header": "Повторная обработка",
  "question": "Как обрабатывается повторный запрос с тем же operationId?",
  "priority": "HIGH",
  "why_it_matters": "Ответ меняет idempotency semantics и test scope.",
  "multiSelect": false,
  "options": [
    {"label": "Вернуть исходный результат", "description": "Новая бизнес-операция не создаётся."},
    {"label": "Вернуть duplicate error", "description": "Повторный запрос отклоняется."},
    {"label": "Создать новую операцию", "description": "operationId не является idempotency key."},
    {"label": "Другое", "description": "Пользователь задаёт иной вариант."}
  ]
}
```

Требования:

- 2–4 evidence-grounded options на вопрос;
- `multiSelect: true` только когда несколько ответов могут одновременно применяться;
- checkbox/multi-select не использовать для взаимоисключающих решений;
- варианты не должны придумывать business rules только ради заполнения UI;
- сначала задавать вопросы с максимальным impact.

## 4. После ответа

Подтверждённый ответ записывается в:

`.agent-runs/<change-id>/clarifications/change-context.json`

Минимально сохрани:

- clarification ID;
- stage;
- вопрос;
- выбранный ответ/ответы;
- `source = USER_CONFIRMED`;
- timestamp/version входных artifacts, если доступно;
- affected requirements/findings/risks/tests/mismatches/failures;
- status `RESOLVED_DURING_ANALYSIS`.

После сохранения:

- invalid/obsolete assumption удалить;
- пересчитать affected выводы;
- не задавать тот же вопрос downstream;
- в финальный unresolved scope выводить только реально незакрытые вопросы.

## 5. Когда ставить WAITING_INPUT

`WAITING_INPUT` нужен, если пользователь не ответил на material question и без ответа нельзя безопасно финализировать stage.

LOW/TRIVIAL non-blocking clarification, не влияющий на expected behavior/decision текущего stage, не должен прерывать flow. Его можно сохранить как follow-up, но нельзя превращать неизвестный ответ в `ASSUMPTION`.

## 6. Stage-specific правила

### spec-review

Спрашивать о material ambiguities/contradictions до формирования финального finding. Resolved issue не оставлять как активный finding.

### risk-analysis

Спрашивать недостающие факты об impact, blast radius, controls и recoverability. Не просить пользователя назначать priority — priority пересчитывает skill.

### test-model-impact-analysis

Спрашивать при реальной неоднозначности `KEEP/UPDATE/OBSOLETE/ADD/REVIEW` или выборе между обновлением существующего coverage и созданием нового.

### test-design

Спрашивать только если ответ меняет expected behavior/test scope. Не спрашивать очевидный выбор test-design technique, который следует из requirement.

### spec-vs-code

Спрашивать о возможных intentional deviations, feature flags, environment/version rules. После ответа всё равно повторно проверить code evidence.

### test-failure-analysis

Спрашивать недостающие факты о стенде, test data, dependency/incident и observed symptoms. Не перекладывать root-cause classification на пользователя.

## 7. Финальные artifacts

В `.md/.json` аналитического stage показывай:

- сколько clarification questions было задано;
- сколько закрыто во время анализа;
- сколько осталось unresolved;
- IDs использованных `USER_CONFIRMED` answers.

Resolved questions не дублируй как активные issues; достаточно traceability на Change Context.
