---
name: final-conformance-check
description: "Перед QA verdict проверяет, не изменились ли approved spec/code/tests после предыдущих validations; при delta выполняет targeted revalidation вместо полного повторного анализа."
priority: 190
---

# Final Conformance Check

## Главный вопрос

Относится ли будущий QA verdict именно к текущим версиям specification, implementation и approved test artifacts?

## Inputs

- current approved analyst artifacts;
- latest implementation revision/diff evidence;
- latest approved manual/autotest artifacts;
- latest execution results;
- previous `spec-vs-code`, `spec-vs-tests`, coverage artifacts;
- recorded version/revision markers, если доступны.

## Алгоритм

1. Сравнить текущие source revisions с теми, которые использовались предыдущими validation stages.
2. Если изменений нет → `CONFORMANCE_UNCHANGED`.
3. Если изменена spec → invalidation начиная со `spec-review`; не выполнять локальную «починку» этого stage.
4. Если изменён code после `spec-vs-code` → определить delta impact и повторить targeted spec-vs-code для затронутых requirements.
5. Если изменены approved tests после `spec-vs-tests` → targeted spec-vs-tests/coverage recheck.
6. Если bugfix изменяет ранее протестированное поведение → проверить наличие relevant retest result.
7. Не требовать полный repo rescan, если delta однозначно ограничена.

## Findings

Используй: `CONFORMANCE_UNCHANGED`, `SPEC_CHANGED`, `CODE_CHANGED_AFTER_VALIDATION`, `TESTS_CHANGED_AFTER_VALIDATION`, `MISSING_RETEST`, `UNKNOWN_REVISION`, `TARGETED_REVALIDATION_REQUIRED`.

## Human Gate

Если обнаружена delta, способная изменить verdict, перед финальным verdict нужен Human review результатов revalidation.

## Definition of Done

Есть evidence, что final verdict опирается на актуальные source revisions, либо flow явно возвращён на нужный upstream stage.
