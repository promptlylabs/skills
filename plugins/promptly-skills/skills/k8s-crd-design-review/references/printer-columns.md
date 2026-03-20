# kubectl printer columns (reference)

Printer columns are operator UX. They should highlight readiness and key intent fields.

## Recommended patterns

- Readiness signal: usually derived from `status.conditions[?(@.type=="Ready")].status`
- Human-friendly status summary: `status.conditions[?(@.type=="Ready")].message` or a dedicated `status.message`
- 1–3 key `spec` intent fields

## Example CRD fragment

```yaml
spec:
  additionalPrinterColumns:
    - name: Ready
      type: string
      jsonPath: .status.conditions[?(@.type=="Ready")].status
      description: Whether the resource is Ready
    - name: Status
      type: string
      jsonPath: .status.conditions[?(@.type=="Ready")].reason
      description: Readiness reason
    - name: Message
      type: string
      jsonPath: .status.conditions[?(@.type=="Ready")].message
      description: Readiness message
    - name: Mode
      type: string
      jsonPath: .spec.mode
      description: Requested operating mode
```

## Review checklist

- Do columns avoid duplicating `AGE`?
- Are jsonPaths valid for the schema produced by the CRD?
- Do columns remain stable across versions (avoid renaming if possible)?

