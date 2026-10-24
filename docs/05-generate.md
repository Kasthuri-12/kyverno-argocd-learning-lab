# 5. Generate policies

A generate rule creates other resources when something happens, such as a new namespace.

Policies:
- [default-deny-networkpolicy.yaml](../policies/generate/default-deny-networkpolicy.yaml): a deny-all NetworkPolicy in every new namespace.
- [default-resourcequota.yaml](../policies/generate/default-resourcequota.yaml): a ResourceQuota in namespaces labelled `managed-quota=true`.
- [rbac-for-generate.yaml](../policies/generate/rbac-for-generate.yaml): lets Kyverno's background controller create those objects. Apply it first. Without it, generation can fail with a "forbidden" error.

## `synchronize: true`
Kyverno keeps the generated resource in step with the policy. If someone edits or deletes it, Kyverno puts it back. Deleting the trigger namespace removes the generated objects with it.

## Lab
```bash
kubectl apply -f policies/generate/rbac-for-generate.yaml
kubectl apply -f policies/generate/default-deny-networkpolicy.yaml
kubectl apply -f policies/generate/default-resourcequota.yaml

kubectl create namespace team-a
kubectl get networkpolicy -n team-a              # default-deny appears

cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: team-b
  labels:
    managed-quota: "true"
EOF
kubectl get resourcequota -n team-b              # default-quota appears
```

### See synchronize working
```bash
kubectl delete networkpolicy default-deny -n team-a
kubectl get networkpolicy -n team-a              # recreated within moments
```

## Notes
- A NetworkPolicy only takes effect if the cluster's network plugin enforces it. minikube's default plugin may not, so you will see the object without any blocking.
- A generate rule fires when a resource is created or updated. Existing namespaces are not touched unless you opt in to generating for existing resources.
- Kyverno's own namespaces are excluded in the NetworkPolicy example so a deny-all rule can't cut off the system.

## Clean up
```bash
kubectl delete namespace team-a team-b
kubectl delete clusterpolicy default-deny-networkpolicy default-resourcequota
kubectl delete -f policies/generate/rbac-for-generate.yaml
```
