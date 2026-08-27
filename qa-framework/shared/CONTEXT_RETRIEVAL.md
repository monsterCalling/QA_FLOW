# Selective Context Retrieval

## Цель

Минимизировать контекст без потери доказательств.

## Уровень 1 — Exact retrieval

Ищи точные identifiers из `change-signals.json`: requirement ID, endpoint, topic/event, entity/table, status, service/class, test-model anchor, Zephyr key.

## Уровень 2 — Catalog matching

Сопоставляй concerns/tags change с `qa-context/context-catalog.yaml`.

## Уровень 3 — Controlled expansion

Если exact/catalog retrieval недостаточен, используй lexical/semantic search только в релевантной области. Читай найденный section/range, а не весь каталог.

## Stop rule

Если после контролируемого поиска нет authoritative evidence, зафиксируй `UNKNOWN`; не компенсируй отсутствие информации полным recursive scan.

## Stage context

Перед запуском каждого stage создай/обнови `.agent-runs/<change-id>/context/<stage>.context-manifest.json`.
