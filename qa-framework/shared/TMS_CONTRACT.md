# TMS Contract

## Principle

QA flow does not push to TMS and does not rewrite approved test semantics.

Sources for export are split:

- `new-test-cases.json` → CREATE;
- `test-case-updates.json` → UPDATE / PROPOSE_OBSOLETE.

Pipeline:

`approved test artifacts → pre-export-validation → export-plan.json → external IDE plugin`.

## Update precision

UPDATE must reference exact existing identity and contain field/step patches with before/after guards. Export layer may transform structure to target API fields, but may not invent or reinterpret test content.

## Safety

- UPDATE without confirmed TMS key is blocked;
- mismatch of `before` value = conflict;
- PROPOSE_OBSOLETE is never automatic delete;
- failed operation does not update `tms-mapping.json`.
