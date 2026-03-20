---
name: k8s-crd-design-review
description: |
  Use when reviewing Kubernetes CustomResourceDefinition (CRD) YAML (full manifests or diffs) as the compiled API contract. Topics: schema/validation (3-step hierarchy: schema → CEL → webhooks only if necessary), spec/status boundaries, conditions/observedGeneration conventions, object-reference modeling resource relationships: one-to-one, one-to-many), SSA/GitOps list semantics, printer columns, and compatibility/migration risk (breaking changes, versioning, rollout). Not for Kubebuilder Go-type selection; use kubebuilder-api-design instead.
---

<!--
Based on k8s-crd-design-review by ConfigButler:
https://github.com/configbutler/skills
-->

# Kubernetes CRD Design Review

Perform a deterministic design + contract review for Kubernetes CRDs (the generated CRD YAML is the compiled API contract).

## Inputs

Accept at least one of:

- CRD YAML (full manifest) or a CRD diff
- A short description of intended API semantics and controller behavior

If key info is missing, ask for it before concluding compatibility/migration:

- Whether a controller exists, what it owns, and whether it writes `status`
- Whether this is a new API or a change to an existing API
- Served versions, storage version, and existing clients
- Whether objects already exist in clusters (migration needed?)
- Any GitOps/SSA constraints (patch strategy, desired stable identities)

## Workflow (always follow this order)

### 1) Identify scope

- Identify `group`, `kind(s)`, `version(s)`, and whether this is **new** vs **change**.
- Identify controller existence and ownership boundaries.
- If reviewing Go types, confirm which generated CRD YAML(s) correspond to them.

### 2) Contract integrity checks (spec/status + controller operability)

- **Spec vs status boundary**
  - `spec` = user intent / desired state.
  - `status` = controller-observed state.
  - Flag lifecycle/state-machine fields in `spec` if the controller owns transitions.
- **Require `subresources.status` when a controller exists and writes status**
- **Conditions + `observedGeneration`**
   - Recommend `status.conditions` and `status.observedGeneration` (Kubernetes conventions; critical for tooling/GitOps correctness).
   - Model Conditions as a **map keyed by `type`**, not a chronological list: prefer schema markers (`x-kubernetes-map-type: map` with `x-kubernetes-list-map-keys: [type]`) for SSA/GitOps safety.
   - Use **state-style Condition types** (adjectives/past tense: `Ready`, `Degraded`, `Succeeded`; avoid transition names or phases for new APIs).
   - Include one high-signal **summary condition** (`Ready` for long-running; `Succeeded` for bounded execution).
   - Ensure each Condition has semantic `True`/`False`/`Unknown` values with consistent meaning.
   - Remember: `status` updates via `/status` subresource use separate RBAC.

See [`./references/conditions-and-status.md`](./references/conditions-and-status.md) for deeper semantics.

### 3) Schema correctness & validation (prevent invalid stored objects)

Use a **3-step validation hierarchy**: always exhaust each level before moving to the next.

#### Step 1: Schema validation (required)
Review the OpenAPI v3 schema (prefer the generated CRD YAML/diff):

- Required fields for true invariants
- Enums for constrained strings
- Defaulting and nullable behavior
- Type constraints, patterns, min/max bounds
- Structural schema: ensure `spec.preserveUnknownFields: false` for strict validation and automatic version conversion
- **Object references & relationships:** When a field refers to another Kubernetes object, use structured references (`fooRef` / `fooRefs`) per conventions. Name-only references (`fooName` as string) acceptable only for existing APIs, not new ones. Watch for cross-namespace references (security boundaries) and spec/status leakage. 
  - Use an object with `group`, `kind`, and `name`
      - add defaults and enum constraints for `group` and `kind`
      - require `name`: always
      - you MAY (very unlikely) add `uid`: UID is assigned by the API server and is not user-friendly or stable for config. It changes if the object is deleted/recreated, therefore you should not use them in spec, but only for reporting on the status field.

In Kubernetes APIs, users reference objects by name + namespace (or just name when same-namespace). The UID belongs in status/observed state, not desired state.Do only do this if you have a real requirement, name is the 'normal' way to reference a resource.
  - Omit `namespace` unless you explicitly allow cross-namespace references.
  - Avoid `apiVersion` in references.
  - `dependsOn` is an advisable (community) deviation from the `fooRefs` advise. Use it when you explicitly model a dependency graph and your controller implements full DAG semantics (readiness definition, scope rules, cycle detection). See [`./references/object-references.md`](./references/object-references.md).
  - Use `parentRef` only for resources your operator directly manages; do not use it to model general relationships.
  - See [`./references/object-references.md`](./references/object-references.md) for schema examples and deeper guidance.

#### Step 2: CEL validation (before webhooks)
When cross-field invariants or complex constraints cannot be expressed with basic OpenAPI rules, use CEL (`x-kubernetes-validations`):

- Cross-field invalid combinations (e.g., "field A only allowed if field B is set")
- Exactly-one-of constraints
- Numeric range relationships between fields
- Enum dependencies

**Best practice:** Write minimal, targeted rules with clear error messages. CEL is stateless, auditable, and version-safe—always prefer it to webhooks.
See [`./references/validation-and-cel.md`](./references/validation-and-cel.md) for examples.

#### Step 3: Webhooks (only if Steps 1–2 are insufficient)
Only recommend webhooks when schema and CEL cannot express the constraint. This is a significant operational decision. **Always double-check first:**

- Can this be expressed with required fields, enums, or patterns?
- Can this be expressed with CEL (stateless, auditable, version-safe)?
- Is the webhook truly necessary, or is the controller solving it better?
- Webhooks add latency, availability risk, and debugging complexity—operational costs often exceed benefits.

If a webhook is necessary:
- **Conversion webhooks:** Use only if structural schema conversion insufficient.
- **Validation/mutation webhooks:** Configure with explicit timeouts, failurePolicy, and namespaceSelectors.
- Validate webhook availability and latency in your rollout plan.

See [`./references/versioning-and-migrations.md`](./references/versioning-and-migrations.md) for conversion strategy and [`./references/review-template.md`](./references/review-template.md) for operational checklist.

### 4) GitOps/SSA ergonomics

Focus on patchability and stable diffs:

- List semantics for arrays of objects
   - If items have stable identity (e.g., `name`, `id`), prefer map-like lists (`x-kubernetes-list-type: map` with `x-kubernetes-list-map-keys`).
   - Identify ordering sensitivity and full-array replacement hazards.

See [`./references/list-semantics-gitops-ssa.md`](./references/list-semantics-gitops-ssa.md).

### 5) Operator UX (kubectl)

Review/add `additionalPrinterColumns` for operator-facing UX:
- Ready / health signal
- Status message / reason
- Key spec fields
- Never duplicate `AGE` (already shown by kubectl).

See [`./references/printer-columns.md`](./references/printer-columns.md).

### 6) Compatibility & migration impact (mandatory)

Always include an explicit compatibility assessment:

- Classify change as non-breaking vs potentially breaking.
- Look beyond removals: tightening validation, type changes, list semantic changes, defaulting changes, semantic behavior shifts.
- If version evolution is involved: plan served versions, conversion webhooks, storage migration, and deprecation playbook.

See [`./references/versioning-and-migrations.md`](./references/versioning-and-migrations.md).

### 7) Synthesize output

Follow the output template structure below:
- Rank risks + explain impact.
- List actionable changes with snippets.
- Provide PR-sized improvement plan.
- Include the PR review template (use [`./references/review-template.md`](./references/review-template.md) as the canonical template).

## Output format (always use this template)

### What’s good

- …

### Top risks (ranked)

1. **…** — why it matters: …
2. **…** — why it matters: …
3. **…** — why it matters: …

### Recommended changes (actionable)

- **Change:** …
  - **Why:** …
  - **Snippet:**
    ```yaml
    # ...
    ```

### Compatibility & migration impact (mandatory)

- **Breaking?** Yes/No
- **Why:** …
- **If breaking or risky:**
  - **Migration / deprecation steps:**
    1. …
    2. …
  - **Rollout plan checklist:**
    - [ ] …
    - [ ] …
  - **Versioning notes:** served versions, storage version, conversion considerations

### Minimal improvement plan (PR-sized)

1. …
2. …
3. …

---

**Template reference:** Use [`./references/review-template.md`](./references/review-template.md) as the canonical template. Adapt to context while preserving section headings.
