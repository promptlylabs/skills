# Conditions and status (reference)

Use Kubernetes conventions for controller-observed state.

## Key recommendations

- Prefer `status.conditions` for readiness/health signals.
- Add `status.observedGeneration` and set it when reconciling.
- If a controller writes `status`, enable `subresources.status`.

Additional recommendations:

- Prefer a single high-signal **summary condition**:
  - `Ready` for long-running resources.
  - `Succeeded` for bounded execution resources.
- Keep **polarity consistent** across Conditions, and choose names that avoid double-negatives (e.g., `Ready`/`Succeeded` are often easier to consume than `Failed`).
- Name Condition `type`s as **states**, not transition phases (e.g., prefer `Active`/`Available`/`Degraded` over `Progressing`/`Scaling`).
- Transition-style names (e.g., `Progressing`) can be acceptable for long-running transitions if treated as an observed state with consistent `True`/`False`/`Unknown` semantics.
- Treat `status.conditions` as a **map keyed by `type`**:
  - Do not append duplicates of the same `type`; update the existing entry.
  - Ordering should not matter.
- Ensure humans and automation can rely on them (e.g., `kubectl wait --for=condition=Ready=true`).

Also prefer Conditions over state-machine style `status.phase` for new APIs.

### Naming conventions (quick)

- Condition `type` values are typically **CamelCase/PascalCase identifiers** (e.g., `Ready`, `Available`, `Degraded`).
- `reason` should also be CamelCase and machine-readable; `message` is human-readable.

### Validating allowed condition `type`s

- Usually **do not** strictly validate the allowed `type` set in the CRD schema; instead **document** the stable condition types your controller uses.
- Hard-enforcing `type` via enum/CEL makes adding a new condition type later a **breaking change** (the API server may reject new values).

## CRD fragment: enable status subresource

```yaml
spec:
  subresources:
    status: {}
```

## Status schema fragment (example)

This is a schema-level example. Tailor field names and required/nullable choices to your API.

```yaml
openAPIV3Schema:
  type: object
  properties:
    status:
      type: object
      description: Observed state of the resource.
      properties:
        observedGeneration:
          type: integer
          format: int64
          description: The generation observed by the controller.
        conditions:
          type: array
          description: Standard status conditions.
          # Conditions are treated as a logical map keyed by `type`.
          # Make that explicit for server-side apply / GitOps patchability.
          x-kubernetes-list-type: map
          x-kubernetes-list-map-keys:
            - type
          items:
            type: object
            required: [type, status, lastTransitionTime, reason]
            properties:
              type:
                type: string
                description: Condition type (CamelCase identifier, e.g., Ready).
              status:
                type: string
                enum: ["True", "False", "Unknown"]
              lastTransitionTime:
                type: string
                format: date-time
              reason:
                type: string
                description: CamelCase machine-readable reason.
              message:
                type: string
                description: Human-readable details.

```

## Condition types

- Include at least one high-signal condition such as `Ready`.
- Add domain-specific conditions only when they provide meaningful operator value.

## Common review flags

- Readiness/lifecycle fields placed in `spec` but transitioned by the controller.
- Missing `observedGeneration` (tooling cannot tell whether status is current).
- Missing `subresources.status` (RBAC/patch semantics and concurrency issues).

## Links

Please do read this if you want to know more: Luis Ramirez, SuperOrbital — “Status and Conditions: Explained!” https://superorbital.io/blog/status-and-conditions/
