# CRD design review template (copy/paste)

Use this template as a PR review comment. Keep headings intact.

## Scope

- **API(s):** <group>/<kind> (<versions>)
- **Change type:** new API | change to existing API
- **Controller:** exists? yes/no; writes status? yes/no; ownership summary

## What’s good

- …

## Top risks (ranked)

1. **…** — why it matters: …
2. **…** — why it matters: …
3. **…** — why it matters: …

## Contract integrity (spec/status, status subresource, Conditions)

- spec/status boundary:
  - …
- `subresources.status`:
  - …
- Conditions + `observedGeneration`:
  - …

## Schema correctness & webhooks (validation, defaults, CEL)

- Required/enums:
  - …
- Cross-field constraints:
  - …
- Object references & relationships:
  - reference field naming (prefer `*Ref`/`*Refs` over `*Name`), namespace scoping, and any info-leak hazards
- Webhook configuration:
  - conversion webhook presence if serving multiple versions with complex schema differences
  - validation/mutation webhook presence if advanced validation/mutation not covered by OpenAPI/CEL
  - `spec.preserveUnknownFields: false` (structural schema) status

## GitOps/SSA ergonomics (lists, patchability)

- List semantics:
  - …
- Patchability concerns:
  - …

## Operator UX (printer columns)

- …

## Compatibility & migration impact (mandatory)

- **Breaking?** Yes/No
- **Why:** …
- **Migration/deprecation plan:**
  1. …
  2. …
- **Rollout plan checklist:**
  - [ ] …
  - [ ] …
- **Versioning notes:** served versions, storage version, conversion needs

## Recommended changes (actionable)

- …

## Minimal improvement plan (PR-sized)

1. …
2. …
3. …
