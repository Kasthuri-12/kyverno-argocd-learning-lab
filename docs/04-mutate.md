# 4. Mutate policies

A mutate rule changes a resource as it is admitted, so people don't have to remember to add boilerplate.

Policy: [add-managed-by-label.yaml](../policies/mutate/add-managed-by-label.yaml) adds `managed-by: kyverno` to every new Pod that has no `managed-by` label.

## Anchors in a mutate patch
| Anchor | Meaning |
|---|---|
| `+(key)` | Add the key only if it is absent |
| `=(key)` | Condition: apply only if the key exists |
| `(key)` | Condition: apply only if the key matches |

## Lab
```bash
kubectl apply -f policies/mutate/add-managed-by-label.yaml

kubectl run demo --image=nginx:1.27
kubectl get pod demo --show-labels               # includes managed-by=kyverno

kubectl run own --image=nginx:1.27 --labels=managed-by=me
kubectl get pod own --show-labels                # keeps managed-by=me, because of +()
```

## Mutate, then validate
Mutation runs before validation. A mutate rule that adds a default can make a validate rule pass, for example adding a `team` label so `require-team-label` is satisfied. Use it for sensible defaults, and use validation for things people must decide themselves.

## Notes
- Mutation applies to new and updated resources. Mutating existing ones needs `mutateExistingOnPolicyUpdate` and extra RBAC.
- Keep mutations small and predictable. Silent changes confuse people when they inspect their own manifests.

## Clean up
```bash
kubectl delete pod demo own
kubectl delete clusterpolicy add-managed-by-label
```
