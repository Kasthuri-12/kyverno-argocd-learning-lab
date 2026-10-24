# 3. Validate policies

A validate rule checks a resource against a pattern and allows or blocks it.

## Policies in this repo
| File | Rule |
|---|---|
| [disallow-latest-tag.yaml](../policies/validate/disallow-latest-tag.yaml) | Images need an explicit tag, and it can't be `latest` |
| [require-requests-limits.yaml](../policies/validate/require-requests-limits.yaml) | CPU and memory requests and a memory limit are required |
| [require-team-label.yaml](../policies/validate/require-team-label.yaml) | Pods need a `team` label |
| [disallow-privileged-containers.yaml](../policies/validate/disallow-privileged-containers.yaml) | No `privileged: true` containers |

The second one enforces the right-sizing habits from cost optimization: a pod without requests can't be scheduled sensibly.

## Pattern syntax
| Symbol | Meaning |
|---|---|
| `"?*"` | Any non-empty value |
| `"*:*"` | Anything containing a colon |
| `"!*:latest"` | Anything that does not end in `:latest` |
| `=(field)` | Check this field only if it exists |
| a list under `containers:` | The pattern applies to every container |

## Lab: Audit first
```bash
kubectl apply -f policies/validate/disallow-latest-tag.yaml
kubectl get clusterpolicy

kubectl run bad-latest --image=nginx:latest      # allowed in Audit mode
kubectl run good --image=nginx:1.27

kubectl get policyreport -A                      # also: kubectl get polr -A
kubectl describe policyreport -n default | head -40
```
The report lists a `fail` result for `bad-latest` and a `pass` for `good`. Reports can take a short while to appear.

## Lab: Enforce
```bash
kubectl patch clusterpolicy disallow-latest-tag --type merge \
  -p '{"spec":{"validationFailureAction":"Enforce"}}'

kubectl run bad2 --image=nginx:latest
```
The request is rejected, and the error includes the policy name, the rule and your message. Existing pods are not deleted: policies act on requests, and reports cover what is already there.

## Controllers are covered too
Kyverno generates matching rules for Deployments, StatefulSets, Jobs and other pod controllers from a Pod rule. A Deployment with a `latest` image is rejected under Enforce, before any pod is created.

## Clean up
```bash
kubectl delete pod bad-latest good --ignore-not-found
kubectl delete clusterpolicy disallow-latest-tag
```

## Reading: verify image signatures
A `verifyImages` rule only lets signed images run. This is illustrative: it needs a real image, a cosign public key and network access to the registry.
```yaml
rules:
- name: verify-signature
  match:
    any:
    - resources:
        kinds: [Pod]
  verifyImages:
  - imageReferences:
    - "registry.example.com/myteam/*"
    attestors:
    - entries:
      - keys:
          publicKeys: |-
            -----BEGIN PUBLIC KEY-----
            <your cosign public key>
            -----END PUBLIC KEY-----
```
