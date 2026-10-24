# 9. The newer CEL-based policy types

Recent Kyverno releases add a second policy API, `policies.kyverno.io/v1`. The official [kyverno/policies](https://github.com/kyverno/policies) library now uses it for its samples.

| New kind | Replaces the idea of |
|---|---|
| `ValidatingPolicy` | `validate` rules |
| `MutatingPolicy` | `mutate` rules |
| `GeneratingPolicy` | `generate` rules |
| `DeletingPolicy` | cleanup policies |
| `ImageValidatingPolicy` | `verifyImages` rules |

Conditions are written in **CEL** (Common Expression Language), the same language Kubernetes' built-in `ValidatingAdmissionPolicy` uses, instead of Kyverno's YAML patterns.

## What the library README says
- The samples need Kyverno 1.17 or later, and individual policies may need more.
- The old `ClusterPolicy`, `Policy`, `ClusterCleanupPolicy` and `CleanupPolicy` samples were removed from the library.
- Policies in the library use `validationActions: [Audit]` so they don't block anything until you change them.

## A side-by-side example
Old style (this lab's [require-team-label.yaml](../policies/validate/require-team-label.yaml)): the label must exist and be non-empty.
```yaml
validate:
  pattern:
    metadata:
      labels:
        team: "?*"
```
New style ([examples/validating-policy-cel.yaml](../examples/validating-policy-cel.yaml)):
```yaml
spec:
  validationActions: [Audit]
  matchConstraints:
    resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["CREATE", "UPDATE"]
      resources: ["pods"]
  validations:
  - expression: "has(object.metadata.labels) && 'app' in object.metadata.labels"
    message: "An app label is required."
```

## Which should you learn?
1. Learn the classic rule model first (this lab). Most tutorials, blog posts and existing clusters use it, and it teaches the concepts: match, validate, mutate, generate, Audit and Enforce.
2. Then read the new API's documentation. The ideas carry over, and only the syntax of the conditions changes.
3. Check what your cluster supports before choosing:
```bash
kubectl get crd | grep -E 'kyverno.io|policies.kyverno.io'
helm list -n kyverno                  # chart and app version
```
If the `ClusterPolicy` CRD is missing or marked deprecated in your Kyverno version, use the new kinds.

The 2023 tutorial this lab started from installs Kyverno 1.8 and uses `ClusterPolicy` only, so its install commands are out of date for current releases.
