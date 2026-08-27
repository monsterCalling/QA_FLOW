---
name: pre-export-validation
description: "Проверяет новые test cases и patch-plan существующих cases как единый effective test set перед подготовкой TMS export plan."
priority: 85
---

# Pre-Export Validation

## Role

Докажи, что TMS handoff не потеряет coverage, identity, atomicity и approved semantics.

## Required inputs

- approved `new-test-cases.json` + corresponding MD;
- approved `test-case-updates.json` + corresponding MD;
- exact current content existing cases targeted by UPDATE;
- approved test-design;
- approved test-model-impact;
- approved test-model update;
- project test-case profile;
- configured TMS mapping;
- `PRE_EXPORT_CONTRACT.md`.

## Rule: do not repair here

Validation stage не исправляет test cases/patches. При проблеме верни точный artifact/item upstream.

## 1. Version / approval integrity

Убедись, что оба test artifacts относятся к approved test-design revision. Stale artifact → `BLOCKED: STALE_APPROVED_ARTIFACT`.

## 2. New-test integrity

Проверь:

- required profile fields;
- unique local IDs;
- `tms_key = null`;
- atomicity PASS;
- no duplicate existing identity/semantic duplicate;
- JSON/MD same approved revision.

## 3. Update-plan integrity

Для каждого UPDATE:

- exact existing identity найден;
- current body доступен;
- TMS key confirmed when external UPDATE is intended;
- `fields_affected` соответствует patch operations;
- каждый patch имеет exact field_path/step_no when applicable;
- `before` совпадает с current source content;
- `after` соответствует approved spec/design;
- source refs существуют;
- нет unrelated rewrites.

Несовпадение `before` → `BLOCKED: UPDATE_BASELINE_CONFLICT`.

## 4. Split reconciliation

Каждый `new_case_request` из maintainer должен быть:

- реализован соответствующим case в `new-test-cases`, либо
- explicitly resolved/withdrawn с evidence.

Иначе `BLOCKED: SPLIT_CASE_MISSING`.

## 5. Build effective test set

Логически вычисли:

`existing cases`  
`+ approved UPDATE patches`  
`- PROPOSE_OBSOLETE only if exclusion/decision permits for coverage calculation`  
`+ new test cases`.

Не изменяй исходные artifacts при построении модели.

## 6. Coverage

Проверь effective set:

`Requirement → Risk → Test Condition → Effective Manual Test`.

Каждый active test condition должен быть COVERED или иметь approved intentional exclusion.

CRITICAL/HIGH MANDATORY risk без coverage → BLOCKED.

Checklist не учитывается как independent coverage.

## 7. TMS compatibility

Проверь mapping отдельно для:

### CREATE
Все required fields `new-test-cases` должны иметь target mapping.

### UPDATE
Каждый `field_path`/step component из patch plan должен быть поддержан plugin mapping. Unsupported → BLOCKED/MAPPING_ERROR before handoff.

## 8. Action candidates

Подготовь candidates:

- CREATE — from `new-test-cases`;
- UPDATE — from `test-case-updates` action UPDATE;
- PROPOSE_OBSOLETE — from update plan;
- SKIP — known unchanged existing cases only for summary/efficiency.

## Outputs

`pre-export-validation.md/json` with:

- integrity.new_cases;
- integrity.updates;
- baseline_conflicts;
- split_reconciliation;
- effective_coverage;
- tms_compatibility;
- action_candidates;
- blocking_issues;
- status PASSED/BLOCKED.

## Definition of Done

PASS означает, что CREATE bodies и UPDATE patches могут быть переданы plugin без скрытой перегенерации, а effective coverage после их применения остаётся достаточным.
