# 6. Reports, exceptions and the Kyverno CLI

## Policy reports
Audit results and background scans are stored as reports:
```bash
kubectl get policyreport -A            # namespaced resources (short name: polr)
kubectl get clusterpolicyreport        # cluster-scoped resources (short name: cpolr)
kubectl describe polr -n default
```
Each entry has the policy, the rule, the resource and `pass`, `fail`, `warn`, `error` or `skip`. Reports turn Audit mode into a worklist: fix the failures, then switch the policy to Enforce.

## Policy exceptions
Sometimes one workload has a good reason to break a rule. A `PolicyException` allows that without weakening the policy for everyone. See [examples/policy-exception.yaml](../examples/policy-exception.yaml).

Exceptions are off by default. Enable them when installing Kyverno, and check the value names for your chart version:
```bash
helm upgrade kyverno kyverno/kyverno -n kyverno \
  --set features.policyExceptions.enabled=true \
  --set features.policyExceptions.namespace='*'
```
Then:
```bash
kubectl create namespace legacy
kubectl apply -f policies/validate/disallow-latest-tag.yaml
kubectl patch clusterpolicy disallow-latest-tag --type merge -p '{"spec":{"validationFailureAction":"Enforce"}}'
kubectl run old --image=nginx:latest -n legacy          # blocked
kubectl apply -f examples/policy-exception.yaml
kubectl run old --image=nginx:latest -n legacy          # allowed now
```
Treat exceptions as reviewed, temporary and owned by someone. Restrict which namespaces may create them.

## Kyverno CLI: test policies without a cluster
The CLI evaluates policies against YAML files on your laptop, so you can learn and run checks in CI with no cluster.
```bash
brew install kyverno
kyverno version
```

Try a policy against sample resources:
```bash
kyverno apply policies/validate/disallow-latest-tag.yaml --resource tests/resources.yaml
```
The output lists a pass or fail for each resource and rule.

Run the repo's tests, which state the result you expect for each resource:
```bash
kyverno test tests/
```
[tests/kyverno-test.yaml](../tests/kyverno-test.yaml) lists the expected results, and [tests/resources.yaml](../tests/resources.yaml) holds a good pod, a pod with the `latest` tag, a pod with no tag and a pod with no resources. The test file format has changed between CLI versions. If `kyverno test` rejects it, check the CLI documentation for the format your version expects.

## Use it in CI
Run `kyverno test` on every pull request that changes `policies/`. A policy change that no longer passes its tests fails the check before it reaches Argo CD.
