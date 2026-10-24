# 7. Argo CD basics

## GitOps in one paragraph
Git holds the desired state of the cluster. A controller inside the cluster keeps comparing the cluster with Git and fixes any difference. Changes happen through pull requests, and a rollback is a `git revert`.

## Argo CD concepts
| Term | Meaning |
|---|---|
| Application | A custom resource: "take this path from this Git repo and deploy it to this cluster and namespace" |
| Source | The Git repo, path and revision, or a Helm chart repo |
| Destination | The cluster and namespace to deploy to |
| Sync status | `Synced` if the cluster matches Git, `OutOfSync` if not |
| Health | Whether the deployed resources are working |
| Auto-sync | Argo CD applies Git changes without a manual click |
| Self-heal | Argo CD reverts manual changes made in the cluster |
| Prune | Argo CD deletes resources that were removed from Git |

## Install
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd -w          # wait until all pods are Running
```
This installs about seven pods, so use a cluster with enough memory.

## Open the UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Open https://localhost:8080 and accept the self-signed certificate warning. The user is `admin`. Get the first password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### The CLI (optional)
```bash
argocd login localhost:8080 --username admin --insecure
argocd app list
```

## Lab: deploy a sample app
```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
EOF
```
In the UI the app shows `OutOfSync`, because auto-sync is off. Click **Sync**, then see the resource tree turn `Healthy`.

### See self-heal
1. Edit the Application and add `automated: {prune: true, selfHeal: true}` under `syncPolicy`.
2. Scale the Deployment by hand: `kubectl scale deploy guestbook-ui -n guestbook --replicas=3`.
3. Watch Argo CD set it back to the replica count in Git.

## Clean up
```bash
kubectl delete application guestbook -n argocd
kubectl delete namespace guestbook
```
