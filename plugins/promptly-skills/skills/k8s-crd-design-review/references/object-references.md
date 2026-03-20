# Object references & relationships (reference)

This reference summarizes Kubernetes API conventions for fields that point at other objects.

## Scope: Configuration References vs. Lifecycle References

This document focuses on **configuration references** — fields where users express relationships for operator configuration (e.g., `secretRef`, `configMapRef`, custom cross-resource dependencies).

**Key distinction**:

- **Configuration References** (primary focus): User-authored references in `spec` that express what other objects an operator should use or coordinate with. These are part of the CRD spec and allow users to define relationships for operator behavior.
  - Examples: `connectionSecretRef`, `serviceAccountRef`, `targetRef`, custom dependency coordination
  - These are typically **same-namespace** and express "what should I use?" or "what should I coordinate with?"

- **Lifecycle References**: System-managed relationships set by controllers, typically not authored directly by users in spec.
  - `ownerReferences`: Parent-child relationships where the operator creates and manages child resources
  - These express "I created this and am responsible for it"
  - See the section [Lifecycle References](#lifecycle-references) for details

## Source

Derived from the upstream Kubernetes API conventions (see "Object references" section):
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md

## Configuration References: Key conventions (review checklist)

This section covers the conventions for fields where users reference other objects in their CRD configuration.

### Naming

- If a field refers to another object via a structured reference, use `fooRef` (one-to-one relationship)
- For lists of references, use `fooRefs` (one-to-many relationship)

The suffix `Ref`/`Refs` is intentionally consistent; avoid ad-hoc names like `fooReference`, `fooObject`, `fooTarget`, etc., unless the purpose demands it (e.g. `targetRef`).

### When is name-only (`fooName`) the right choice?

Prefer a structured reference (`fooRef`) by default when you can: being explicit in the schema tends to age better, is friendlier to tooling (e.g., editor plugins that allow to "jump to definition"), and keeps future evolution additive.

Using a plain string name is still **allowed**, but treat it as an optimization for very constrained cases where **the controller can unambiguously infer everything else**. 

`fooName` is ok-ish when the following are all true:

- The reference is **always to exactly one resource type** (controller has a fixed GVR / kind).
- The reference is **same-namespace** (or the referenced object is cluster-scoped), so `namespace` does not need to be expressed.
- You do not need qualifiers like `group`, `kind`, or `fieldPath`.
- You do not expect the reference to evolve into multi-kind or cross-namespace.

If there is any reasonable chance you will later want qualifiers like `group`/`resource`/`kind`, cross-namespace support, multi-kind support, or better UX for "reference browsing", start with `fooRef`. Changing a field from `fooName: string` to `fooRef: object` is typically a **breaking API change**.

### Namespace scope

- For **namespaced** resources, references should normally be **same-namespace**.
- Cross-namespace references are discouraged because namespaces are security boundaries.
  - If you allow them anyway, explicitly document semantics (creation ordering, deletion behavior, permissions), and consider admission-time permission checks or double opt-in.

### Schema shape

- Prefer a structured object over a bare string as it gives your reference the flexibility to evolve (e.g., from single-kind to multi-kind).
- Avoid including `apiVersion` in references. The API conventions recommend letting controllers handle versioning. This is intentionally less explicit: it allows referenced resource schema versions to change over time. Controllers should be able to discover or map the correct version; hard-coding versions in references makes evolution and upgrades more brittle. The only exception is **field references**, where the schema version is required to interpret `fieldPath`.

### Defaults & requiredness (practical guidance)

- Do not default the *identity* (`name`). A reference without a `name` is not actionable.
- Defaulting *qualifiers* like `group` / `kind` **can be a good UX and tooling-friendly design** when the reference is currently single-kind and you want users to be able to write:

  ```yaml
  targetRef:
    name: my-name
  ```

  This keeps the schema explicit enough that tools can infer what is being referenced, while keeping the authoring surface small.

- Treat defaults as part of the API contract:
  - Changing defaults later can be a semantic breaking change.
  - If you anticipate expanding to additional target types in the future, you can keep the existing defaults for backward compatibility and add optional fields/validation to support additional types.

- If a reference is required for the resource to be meaningful, make the reference field required (e.g., require `fooRef` or `fooName` in `spec`).
- Within a `*Ref` object, require the identifying parts: `name` (and `namespace` only if you explicitly allow cross-namespace references). Pick defaults for `group` and `kind` when possible.
- Prefer omitting `namespace` (assume same-namespace) over defaulting it. If you do include `namespace`, treat it as an explicit, security-sensitive choice.

Example (single-kind today, future-proof for later):

```yaml
targetRef:
  properties:
    group:
      type: string
      default: example.com
    kind:
      type: string
      default: GitTarget
      enum: [GitTarget]
    name:
      type: string
  required: [name]
```

## Configuration References: Recommended schemas

This section shows schema patterns for common reference use cases.

### Single-kind reference (controller knows GVR)

For single-kind references, the conventions allow the controller to hard-code qualifiers. If you choose to include qualifiers for clarity or future-proofing, keep them defaulted and constrain them via enum. For example, a field that references a secret:

```yaml
connectionSecretRef:
  properties:
    name:
      type: string
    group:
      type: string
      default: ""
    kind:
      type: string
      default: Secret
      enum: [Secret]
  required: [name]
```

Example usage (still lean due to defaults):

```yaml
spec:
  connectionSecretRef:
    name: my-secret
```

### Multi-kind reference (bounded set of supported types)

Use when the reference can point to more than one type.

```yaml
spec:
  targetRef:
    group: example.com
    kind: Widget
    name: my-widget
```

Notes:

- **Practice vs. spec**: upstream conventions prefer `group` + `resource` + `name`, but most APIs (e.g. Flux and Crossplane) use **GKV** in practice; this guide follows that reality.
- Use `kind` only when your controller has a **predefined, unambiguous mapping** from kind to resource.
- Including `group` avoids ambiguity and helps copy/paste portability.

## Configuration References: Controller behavior guidance

- Assume the referenced object might not exist; surface a clear error via Conditions/Events.
- Validate reference fields before using them as API path segments.
- Do **not** modify the referenced object (avoid privilege escalation vectors).
- Minimize copying values from the referenced object into the referrer (including `status` and Events/logs), to avoid leaking information a user may not have permission to read.

## Configuration References: Common review flags

- Reference fields not suffixed with `Ref`/`Refs`.
- Cross-namespace references without explicit semantics and guardrails.
- Free-form strings used for "references" where a structured schema is expected.
- Status/spec fields that echo data read from the referenced object without a clear, safe rationale.

## Lifecycle References

This section covers relationship types managed by controllers, not users. These are set by operator code, not in user-authored spec.

### ownerReferences: Parent-child lifecycle

`ownerReferences` is a Kubernetes-native mechanism for expressing parent-child lifecycle relationships. It is **system-managed** — set by controllers when they create child resources.

**Use case**: When your operator **creates and manages child resources** (e.g., an Operator creates Deployments, ConfigMaps, Services to implement a larger object).

**Key semantics**:

- **Lifecycle ownership**: Defines that the parent is responsible for the child.
- **Garbage collection**: When the parent is deleted, Kubernetes automatically deletes children (unless `blockOwnerDeletion: true`).
- **Finalizers**: Controllers can use finalizers to clean up before deletion.
- **Not user-configured**: Users do not write `ownerReferences` in spec; controllers set them automatically.

**Example**: A CRD MyDatabase might create a Deployment, a Service, and a ConfigMap as children. The MyDatabase controller sets `ownerReferences` on these objects pointing back to the MyDatabase instance.

**Key distinction from configuration references**: `ownerReferences` expresses "I created this and manage its lifecycle," whereas configuration references express "I need to use or coordinate with this existing object."

## Community patterns

This section is intended to grow over time as additional patterns emerge.

### 1. Dependency graph with `dependsOn`

#### Reviewer guidance (generic CRDs)

`dependsOn` is a **tool-specific orchestration pattern**, not a general reference type. Accept it only if the controller/operator implements **all** of the following:

1. **Defines what "dependency satisfied" means** — typically the referenced object has `status.conditions[].type=Ready` with `status=True`.
2. **Specifies scope rules** — whether references are namespace-scoped, cross-namespace, or cluster-scoped; and if cross-namespace, what permissions/admission controls validate the dependency.
3. **Defines cycle behavior and circular dependency detection** — build and validate a DAG, and report cycles before reconciliation proceeds.
4. **Documents it clearly** — in API docs and operator guide.

#### Dependency graph validation pattern

If implementing `dependsOn`, validate a directed acyclic graph (DAG) at admission time when possible, and surface conflicts via conditions when detected at reconciliation time.

#### Recommended alternative approach

If your CRD does not implement active dependency orchestration:

- **Clear status conditions** — define explicit `Ready`, `Reconciling`, `Stalled`, and failure conditions that reflect your reconciliation state.
- **Explicit readiness checks** — implement controller logic that checks readiness of dependent resources before proceeding; do not rely on implicit field semantics.
- **Distinguish reference types** — configuration references (user-authored in spec), `dependsOn` (orchestration ordering), and `ownerReferences` (parent-child lifecycle). Use configuration references for object pointers, use `dependsOn` only with full DAG behavior, use `ownerReferences` for controller-managed children.

#### Note on cross-kind dependencies and reference type distinction

Flux's `dependsOn` is typically **same-kind** (Kustomization → Kustomization, HelmRelease → HelmRelease). Treat this as a **community pattern**, not a general reference rule, and avoid expanding it without explicit controller behavior and documentation.

**Summary of reference types**:

- **Configuration references** (user-authored in spec): "I need to use or coordinate with this object" (e.g., `secretRef`, `serviceAccountRef`)
- **`dependsOn`** (tool-specific orchestration): "Don't reconcile me until this other resource reaches Ready" (requires explicit controller implementation)
- **`ownerReferences`** (Kubernetes native, controller-managed): "I created this; manage its lifecycle and garbage collection" (set by controllers when they create child resources)

If your use case involves users specifying configuration dependencies, use configuration references. If it involves orchestrating **existing resources** in a specific order, use `dependsOn` with full DAG validation. If it involves an operator that **creates child resources**, use `ownerReferences`.

### References

- **Flux Kustomize/Helm CRD API** (reference for `dependsOn` pattern):
  https://fluxcd.io/flux/components/kustomize/api/v1/

- **Kubernetes ownerReferences** (lifecycle/garbage collection, not ordering):
  https://kubernetes.io/docs/concepts/overview/working-with-objects/owners-dependents/

- **Upstream Kubernetes API conventions** (object references):
  https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
