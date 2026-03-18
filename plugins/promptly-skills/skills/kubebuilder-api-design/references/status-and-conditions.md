# Status and Conditions (Kubebuilder implementation appendix)

This file is intentionally **Kubebuilder/Go-focused**: how to express a good status/conditions contract in Go types + markers so the generated CRD YAML matches what you intended.

Canonical conceptual guidance (naming, semantics, common review flags) lives in:

- [`../k8s-crd-design-review/references/conditions-and-status.md`](../k8s-crd-design-review/references/conditions-and-status.md:1)

## Baseline status shape (Go)

When designing a status for a long-running reconciled resource, a common baseline is:

- `observedGeneration` (controller-owned)
- `conditions` as `[]metav1.Condition` (controller-owned)

### Example: status struct + list semantics markers

```go
import metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"

type FooStatus struct {
  // ObservedGeneration is the `.metadata.generation` last observed by the controller.
  //
  // +kubebuilder:validation:Minimum=0
  // +optional
  ObservedGeneration int64 `json:"observedGeneration,omitempty"`

  // Conditions represents the latest available observations of the resource's state.
  //
  // Implement logical-map semantics (keyed by `type`) so SSA/GitOps can patch individual
  // conditions and you avoid duplicate `type` entries.
  //
  // +listType=map
  // +listMapKey=type
  // +kubebuilder:validation:MaxItems=20
  // +optional
  Conditions []metav1.Condition `json:"conditions,omitempty"`
}
```

Notes:

- The controller should set `status.observedGeneration` when it updates `status`.
- Bound the conditions array with `MaxItems` to prevent unbounded growth.

## Enable the status subresource (root marker)

If a controller writes status, ensure the root type enables the `/status` subresource:

```go
// +kubebuilder:subresource:status
// +kubebuilder:object:root=true
type Foo struct {
  // ...
}
```

This ensures status updates use the correct endpoint and RBAC boundary.

## (Optional) Printer columns for status/conditions

If your API benefits from `kubectl get` surfacing readiness, add printer columns on the root type.

Example (JSONPath varies by preference/tooling):

```go
// +kubebuilder:printcolumn:name="Ready",type=string,JSONPath=".status.conditions[?(@.type=='Ready')].status",description="Ready condition status"
// +kubebuilder:printcolumn:name="Age",type=date,JSONPath=".metadata.creationTimestamp"
```

For review guidance on which columns help operator UX (and common anti-patterns), see:

- [`../k8s-crd-design-review/references/printer-columns.md`](../k8s-crd-design-review/references/printer-columns.md:1)

## Generate and verify

After adjusting Go types/markers:

- Run `make generate` and `make manifests`.
- Inspect the generated CRD YAML and confirm it contains:
  - `spec.versions[*].schema.openAPIV3Schema.properties.status.properties.conditions`
  - `x-kubernetes-list-type: map` and `x-kubernetes-list-map-keys: ["type"]`

Then run a contract review pass using:

- [`../k8s-crd-design-review/SKILL.md`](../k8s-crd-design-review/SKILL.md:1)
