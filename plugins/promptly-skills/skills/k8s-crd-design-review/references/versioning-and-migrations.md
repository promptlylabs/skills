# Versioning and migrations (reference)

Every CRD change must be assessed for compatibility impact and whether users need a migration plan.

## Breaking change taxonomy (common)

Schema and contract breaks:

- Removing a field
- Renaming a field
- Type changes (string → int, object → string, etc.)
- Tightening validation (e.g., new required fields, narrower enums, new CEL rules)
- Changing defaulting behavior
- Changing list semantics (atomic → map, key changes, order meaning)

Behavioral breaks (even if schema looks similar):

- Controller interprets existing values differently
- Reconciliation changes that invalidate old objects
- Status semantics changes that break automation

## Deprecation playbook (field-level)

Use a multi-release approach:

1. Introduce new field(s) alongside old
2. Keep old field(s) working
3. Emit warnings / Conditions / Events indicating deprecation
4. Update docs/examples and clients
5. Eventually remove old field(s) in a major version (or when policy allows)

## Versioning strategies

- Serve multiple versions when you need to evolve the schema without forcing immediate migrations.
- **Structural schema requirement**: Ensure CRDs are marked as `spec.preserveUnknownFields: false`. This forces all fields to be defined in the schema, enabling:
  - Strict API server validation of all fields
  - Automatic conversion between API versions for simple changes
  - Better tooling support (kubectl, OpenAPI generation, etc.)
  - Predictable behavior for clients
- Use conversion webhooks when versions are meaningfully different (structural schema conversion is insufficient)
- For simple additive changes, structural schema can handle conversion automatically

### Preserving unknown fields (advanced topic)

The `spec.preserveUnknownFields` field and `x-kubernetes-preserve-unknown-fields` annotation are **deprecated patterns** that should be avoided in most cases:

**Key context** (per [Kubernetes CRD documentation](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)):
- Starting from `apiextensions.k8s.io/v1`, **pruning is the default behavior** for CRDs with structural schemas
- `spec.preserveUnknownFields` defaults to `false`, meaning unknown fields are pruned (removed) before persistence
- CRDs created in `apiextensions.k8s.io/v1beta1` with `spec.preserveUnknownFields: true` are legacy patterns that continue to work for backward compatibility

**Why avoid `x-kubernetes-preserve-unknown-fields: true`:**
- Introduces security risk: unknown fields can bypass validation and be persisted to etcd (see the [Future of CRDs: Structural Schemas](https://kubernetes.io/blog/2019/06/20/crd-structural-schema/) blog post)
- Conflicts with client-side validation tools (kubectl, IDE tooling)
- Prevents schema evolution and makes schema discovery difficult
- Incompatible with conversion webhooks and defaulting features
- CEL validation rules cannot access preserved unknown fields

**Legitimate use cases** (rarely needed):
- When you intentionally store arbitrary JSON (use a JSON field with explicit type and bounds instead)
- At the root level as a migration aid for very old CRDs: `type: object` + `x-kubernetes-preserve-unknown-fields: true`
- Per-field basis: narrow down to specific fields that genuinely need to accept arbitrary values

**Alternative approach**: Define fields explicitly in the schema. For flexible data:

```yaml
# Instead of: x-kubernetes-preserve-unknown-fields: true
myFlexibleData:
  type: object
  additionalProperties: true
  description: "Arbitrary key-value data"
```

#### Interaction with `x-kubernetes-embedded-resource`

When using `x-kubernetes-embedded-resource: true` (for embedding Kubernetes resources like Pods):

```yaml
foo:
  x-kubernetes-embedded-resource: true
  x-kubernetes-preserve-unknown-fields: true  # optional but often needed
```

- `x-kubernetes-embedded-resource: true` automatically ensures `apiVersion`, `kind`, and `metadata` are validated and not pruned
- Using `x-kubernetes-preserve-unknown-fields: true` with embedded resources allows storage of complete Kubernetes objects without field constraints
- **Important**: Using both together doesn't improve security—the embedded resource still needs validation through other means (validation rules, webhooks, controller logic)
- If you only need a subset of fields from the embedded resource, prefer defining an explicit schema without `x-kubernetes-preserve-unknown-fields: true`

**Reference**: [Kubernetes API Extension Conventions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema) and [Custom Resource Definition Versioning](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)

**Example CRDs:**
- See [`../examples/crd-preserveunknownfields-good.yaml`](../examples/crd-preserveunknownfields-good.yaml) for a proper CRD demonstrating explicit schema and best practices
- See [`../examples/crd-preserveunknownfields-antipattern.yaml`](../examples/crd-preserveunknownfields-antipattern.yaml) for problematic patterns to avoid

## Storage version reminders

- Storage version changes require a plan for existing stored objects.
- Ensure conversion is correct in both directions for all served versions.

## Review checklist

- Is this change safe for existing stored objects?
- If not, is there an explicit migration plan?
- Are rollout steps and communication/deprecation steps defined?

