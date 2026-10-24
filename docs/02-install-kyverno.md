# 2. Install Kyverno

Use a throwaway cluster such as minikube, and check `kubectl config current-context` first.

## Install with Helm
```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
helm search repo kyverno/kyverno --versions | head

helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```
The default install runs one replica of each controller, which is enough for learning. For a real cluster run three replicas of the admission controller so the webhook stays available:
```bash
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set admissionController.replicas=3
```
The value names changed between chart major versions (older charts used `replicaCount`). Check `helm show values kyverno/kyverno` for the chart you installed.

## Check it
```bash
kubectl get pods -n kyverno
kubectl get crd | grep kyverno
kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations | grep kyverno
```
You should see running controller pods, CRDs such as `clusterpolicies.kyverno.io` and `policyreports.wgpolicyk8s.io`, and webhook configurations.

## Resources
Kyverno and its pods take a few hundred MB of memory. Running Kyverno and Argo CD together needs more than a 4 GB minikube; start with `minikube start --cpus=4 --memory=6144` if your machine allows it.

## Uninstall
```bash
helm uninstall kyverno -n kyverno
kubectl delete namespace kyverno
kubectl get crd | grep kyverno         # CRDs can remain after uninstall
```
Delete leftover Kyverno CRDs and webhook configurations only if you are sure nothing else uses them. Stale webhooks can block API requests.
