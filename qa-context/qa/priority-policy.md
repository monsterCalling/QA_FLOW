# QA Finding and Risk Priority Policy

## 1. Назначение

Этот документ задаёт единую шкалу приоритетов для QA findings, risks и clarification questions и определяет, когда QA-flow обязан остановиться, а когда может продолжиться автоматически.

Главный принцип:

> Наличие замечания или риска само по себе не является причиной остановки flow. Решение определяется типом сущности, её приоритетом и влиянием на возможность однозначно реализовать/проверить change.

## 2. Каноническая шкала

Используй только следующие значения `priority`:

- `CRITICAL`
- `HIGH`
- `MEDIUM`
- `LOW`
- `TRIVIAL`

Legacy aliases допустимы только при чтении старых артефактов:

- `P0` → `CRITICAL`
- `P1` → `HIGH`
- `P2` → `MEDIUM`
- `P3` → `LOW`

Новые артефакты должны использовать канонические значения.

## 3. Finding priority

Finding — проблема в требованиях, аналитических артефактах или их согласованности.

### CRITICAL

Использовать, если проблема не позволяет безопасно продолжать без исправления, например:

- требования допускают взаимоисключающее критичное поведение;
- невозможно определить корректный expected behavior критичной операции;
- спецификация допускает финансовую потерю/дублирование или необратимую порчу критичных данных;
- отсутствует критичное security/authorization правило, без которого реализация небезопасна;
- authoritative sources прямо противоречат друг другу по критичному поведению.

`flow_action = HARD_BLOCK`.

### HIGH

Использовать, если проблема способна существенно изменить реализацию или проверку основного business flow, например:

- не определена важная state transition semantics;
- не определено существенное error/retry/idempotency behavior;
- API/Kafka/data contract неоднозначен для важного сценария;
- есть существенная compatibility/security ambiguity;
- разные разумные трактовки приведут к разной бизнес-логике.

`flow_action = HUMAN_REVIEW`.

### MEDIUM

Использовать, если проблема влияет на отдельный сценарий или качество реализации, но основной flow остаётся однозначным и существует безопасный путь продолжения.

`flow_action = HUMAN_REVIEW`.

### LOW

Использовать, если замечание полезно исправить, но оно не меняет основной expected behavior и не мешает проектированию/реализации change.

Примеры:

- minor missing detail, не влияющий на decision;
- локальная терминологическая неясность с очевидным контекстом;
- второстепенная observability/documentation gap.

`flow_action = CONTINUE`.

### TRIVIAL

Использовать для чисто косметических или редакционных замечаний:

- typo;
- formatting;
- naming/style consistency;
- несущественная документационная косметика.

`flow_action = CONTINUE`.

## 4. Finding gate policy

После `spec-review` flow **всегда останавливается для human decision**. Priority определяет тип этого gate, а не факт его наличия.

| Effective result | Действие | Gate type |
|---|---|---|
| есть `CRITICAL` | `HARD_BLOCK` | `BLOCKING_REVIEW` |
| есть `HIGH` или `MEDIUM` | `HUMAN_REVIEW` | `REVIEW_GATE` |
| сработала aggregate escalation | `HUMAN_REVIEW` | `REVIEW_GATE` |
| только `LOW/TRIVIAL` в допустимых пределах | `CONFIRM` | `CONFIRMATION_GATE` |
| findings отсутствуют | `CONFIRM` | `CONFIRMATION_GATE` |

`LOW` и `TRIVIAL` findings не требуют исправления для продолжения и не превращаются в blockers, но финальное решение о переходе из `spec-review` остаётся за человеком.

На `CONFIRMATION_GATE` допустимы минимум два явных действия:

- `APPROVE_AND_CONTINUE`;
- `RETURN_TO_ANALYST`.

Модель не может выбрать `APPROVE_AND_CONTINUE` самостоятельно.

## 5. Aggregate escalation для findings

Несколько мелких проблем могут указывать на системно слабую спецификацию.

По умолчанию повысить **stage effective priority** до `MEDIUM` и потребовать Human Review, не меняя individual priorities, если:

- найдено 5 или более `LOW` findings; или
- найдено 3 или более `LOW` findings в одной business/contract area; или
- совокупность LOW findings делает один requirement фактически неоднозначным.

`TRIVIAL` findings сами по себе aggregate escalation не вызывают.

В artifact обязательно указать `aggregate_escalated=true` и причину.

## 6. Risk priority

Risk — потенциальный failure mode продукта/реализации. Risk не равен finding.

Высокий risk **не означает автоматически**, что аналитика плохая или change надо вернуть аналитику. Он прежде всего задаёт test obligation и необходимость human validation.

### CRITICAL risk

Потенциальные последствия: catastrophic business/financial/data/security impact, массовый отказ, необратимое повреждение критичных данных.

- `flow_action = HUMAN_REVIEW`
- `test_obligation = MANDATORY`

### HIGH risk

Существенный business/data/integration/security/regression impact.

- `flow_action = HUMAN_REVIEW`
- `test_obligation = MANDATORY`

### MEDIUM risk

Ограниченный/recoverable impact или значимый локальный сценарий.

- `flow_action = HUMAN_REVIEW`
- `test_obligation = RECOMMENDED`

### LOW risk

Низкий business impact, легко обнаруживаемый/recoverable сценарий.

- `flow_action = CONTINUE`
- `test_obligation = OPTIONAL`

### TRIVIAL risk

Практически отсутствующий business impact; обычно это не должен быть отдельный risk item. Использовать только если команда явно хочет сохранить наблюдение.

- `flow_action = CONTINUE`
- `test_obligation = NONE`

## 7. Risk gate policy

| Highest risk priority | Действие stage |
|---|---|
| CRITICAL | `HUMAN_REVIEW` |
| HIGH | `HUMAN_REVIEW` |
| MEDIUM | `HUMAN_REVIEW` |
| LOW | `CONTINUE` |
| TRIVIAL | `CONTINUE` |
| risks отсутствуют | `CONTINUE` |

Важно: risk-analysis не должен ставить `HARD_BLOCK` только из-за наличия CRITICAL/HIGH risk. Если анализ риска выявил именно пробел требований, создай/предложи `finding_candidate` и маршрутизируй проблему обратно в `spec-review`.

## 8. Clarification priority

Clarification во время `spec-review` должен задаваться **сразу после обнаружения материальной неопределённости**, а не откладываться до финального отчёта.

Blocking/material clarification нужен, если неизвестный ответ может изменить:

- expected behavior;
- `CRITICAL/HIGH/MEDIUM` finding;
- обязательный test scope;
- API/Kafka/data/state contract;
- retry/idempotency/transaction/error semantics;
- решение о готовности аналитики.

Такие вопросы задаются интерактивными rounds через native `AskUserQuestion`/эквивалент runtime. В одном round: не более 4 связанных вопросов. Для каждого вопроса:

- короткий `header`;
- один конкретный `question`;
- 2–4 evidence-grounded options;
- `multiSelect=false` для взаимоисключающих вариантов;
- `multiSelect=true`/checkboxes только если несколько вариантов могут одновременно быть верны;
- вариант `Другое` допускается, если предложенные варианты не исчерпывающие.

После ответа:

1. сохранить его как `USER_CONFIRMED` в Change Context;
2. пересчитать затронутые findings/priority/scope;
3. пометить вопрос `RESOLVED_DURING_REVIEW`;
4. не включать его в финальный unresolved question scope;
5. не задавать его повторно downstream skills.

`LOW/TRIVIAL` вопросы не должны прерывать анализ: сохранить их как non-blocking follow-up, если ответ не нужен для текущего решения.

## 9. Приоритет и evidence

Приоритет нельзя назначать по ощущениям или только из-за домена приложения.

Каждый `CRITICAL/HIGH/MEDIUM` item должен содержать:

- конкретное возможное последствие;
- affected behavior/scope;
- evidence;
- почему более низкий priority недостаточен.

Если impact неизвестен и от него зависит priority — задать blocking clarification.

## 10. Human override

Человек может явно:

- принять риск и продолжить;
- вернуть аналитику;
- понизить/повысить priority с причиной;
- заблокировать flow независимо от автоматически рассчитанного priority.

Решение сохраняется в `state.json.latest_decision` и имеет приоритет над рекомендацией LLM для текущей версии artifact.
