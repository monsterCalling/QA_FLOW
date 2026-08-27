# Human Gate Contract

## 1. Главное правило

Human Gate является абсолютной остановкой **после того, как gate был активирован**. Модель не может self-approve.

Не каждый configured stage обязан останавливаться безусловно: `spec-review` и `risk-analysis` используют priority-aware gate согласно `qa-context/qa/priority-policy.md` и `qa-framework/qa-flow.yaml`.


## Interactive clarification vs Human Gate

Clarification rounds (`AskUserQuestion`) выполняются **внутри stage до его финализации** и не являются approval. Они могут происходить в `spec-review`, `risk-analysis`, `test-model-impact-analysis`, `test-design`, `spec-vs-code` и `test-failure-analysis`.

После ответа stage обязан пересчитать затронутые выводы и затем всё равно применить configured Human Gate. Пользовательский ответ на clarification нельзя трактовать как `APPROVE`.

## 2. Priority-aware gates

### Findings (`spec-review`)

`spec-review` **всегда заканчивается Human Gate** после interactive clarifications и critic/reconciliation. Priority определяет тип gate:

- `CRITICAL` → `BLOCKING_REVIEW`: hard stop;
- `HIGH/MEDIUM` → `REVIEW_GATE`: человек разбирает существенные проблемы;
- aggregate escalation LOW → `REVIEW_GATE`;
- только `LOW/TRIVIAL` ниже thresholds → `CONFIRMATION_GATE`;
- findings отсутствуют → `CONFIRMATION_GATE`.

На `CONFIRMATION_GATE` controller показывает краткий summary и спрашивает только, можно ли идти дальше. Минимальные действия:

- `APPROVE_AND_CONTINUE`;
- `RETURN_TO_ANALYST`.

LOW/TRIVIAL сохраняются в artifacts, но сами по себе не требуют исправления. Модель не может подтвердить gate сама.

### Risks (`risk-analysis`)

- `CRITICAL/HIGH/MEDIUM` → `WAITING_HUMAN` для validation/acceptance/test focus;
- только `LOW/TRIVIAL` → Human Gate не создаётся и flow продолжается;
- risk сам по себе не является requirement defect и не должен автоматически возвращать change аналитику.

Если risk выявляет requirement gap, маршрут должен идти через finding/spec-review.

## 3. Unconditional Human Gates

Остальные gates, явно заданные в `qa-flow.yaml` через `human_gate`, всегда останавливают flow.

## 4. Решения человека

- `APPROVE` — принять обычный review stage и продолжить.
- `APPROVE_AND_CONTINUE` — явное подтверждение lightweight `CONFIRMATION_GATE` после `spec-review`.
- `ACCEPT_RISK_AND_CONTINUE` — подтвердить awareness риска и продолжить; не означает устранение риска.
- `REJECT` — вернуть stage на доработку с причиной.
- `RETURN_TO_ANALYST` — аналитика требует изменения; flow ждёт обновления analyst artifacts.
- `BLOCK` — остановить flow.
- `REPRIORITIZE` — изменить priority с обязательной причиной; сохранить решение и reason в `state.json.latest_decision`.

## 5. Повтор после изменения аналитики

Если `proposal.md`, `preview.md` или `spec.md/specs/**` изменены после approved/accepted `spec-review`, `qa-flow` обязан:

1. пометить `spec-review` и зависимые downstream stages как `INVALIDATED`;
2. повторить `context-discovery` для delta/context update;
3. повторить `spec-review`;
4. не переносить старый approval автоматически.

## 6. Checklist + test-case changes gate

После `checklist-writer` и stage `test-case-changes` используется единый Human Gate `CHECKLIST_AND_TEST_CASE_CHANGES`. Пользователь видит:

- `checklist.md/json` — краткий набор проверок;
- `new-test-cases.md/json` — только новые полные атомарные cases;
- `test-case-updates.md/json` — только existing cases и exact field/step patches;
- atomicity summary;
- coverage delta / effective coverage summary.

Можно вернуть на correction только new cases, только update plan, checklist или несколько частей. Approval фиксирует все три как согласованный test-change set.

## 7. TMS handoff

После approve test-change set flow выполняет `pre-export-validation`. При PASS создаётся export plan. Фактическая команда выгрузки через IDE plugin выполняется пользователем отдельно; это не self-action QA flow.

`PROPOSE_OBSOLETE` никогда не означает автоматический delete/archive.
