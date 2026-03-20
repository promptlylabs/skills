# List semantics for GitOps / SSA (reference)

Arrays of objects often cause noisy diffs and patch failures because ordering matters and updates can require replacing the entire array.

## Prefer map-like lists when items have stable identity

If each entry has a stable key (e.g., `name`, `id`, `port`), prefer:

- `x-kubernetes-list-type: map`
- `x-kubernetes-list-map-keys: ["<key>"]`

### Example (CRD schema fragment)

```yaml
properties:
  spec:
    type: object
    properties:
      listeners:
        type: array
        x-kubernetes-list-type: map
        x-kubernetes-list-map-keys:
          - name
        items:
          type: object
          required: [name, port]
          properties:
            name:
              type: string
            port:
              type: integer
              minimum: 1
              maximum: 65535
```

## When a list should stay atomic

Keep a list as an atomic array when:

- Ordering is semantically meaningful.
- Items do not have a stable identity.
- The list is small and only ever replaced as a whole.

## Review checklist

- Does each list of objects have a clear identity key?
- Are there duplicate-key hazards that need validation?
- Will SSA/GitOps updates be common and partial?

## Common pitfalls

- Object arrays without map semantics in GitOps-managed APIs (constant churn).
- Using `name` keys that are not truly stable (e.g., derived display names).
- Changing list semantics after objects already exist (compatibility risk).

