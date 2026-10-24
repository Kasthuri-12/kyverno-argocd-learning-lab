# Kyverno and Argo CD Learning Lab

Learn how to enforce security and governance rules on a Kubernetes cluster with **Kyverno** (policy as code) and keep those rules in Git with **Argo CD** (GitOps). The repo has concept guides, a library of working policies, offline policy tests and the Argo CD manifests that tie them together.

The commands and manifests were written from the Kyverno and Argo CD documentation. Check them against the versions you install.

## Learning path

| Step | Guide | You learn |
|---|---|---|
| 1 | [Concepts](docs/01-concepts.md) | Admission control, how Kyverno works, policy anatomy, Audit and Enforce |
| 2 | [Install Kyverno](docs/02-install-kyverno.md) | Helm install, what gets created, how to uninstall cleanly |
| 3 | [Validate](docs/03-validate.md) | Block `latest` tags, missing limits, missing labels and privileged pods |
| 4 | [Mutate](docs/04-mutate.md) | Add defaults automatically |
| 5 | [Generate](docs/05-generate.md) | Create NetworkPolicies and quotas for new namespaces |
| 6 | [Reports, exceptions and CLI](docs/06-reports-exceptions-and-cli.md) | Policy reports, exceptions, and testing policies offline |
| 7 | [Argo CD basics](docs/07-argocd-basics.md) | GitOps, Applications, sync, self-heal |
| 8 | [GitOps for policies](docs/08-gitops-for-policies.md) | Deliver policies with Argo CD, app of apps, sync waves, drift correction |
| 9 | [New policy types](docs/09-new-policy-types.md) | The newer CEL-based policy API and how to choose |

## Learn with or without a cluster
- **No cluster:** read the guides, and use the Kyverno CLI to run `kyverno test tests/` against the sample resources.
- **Small cluster (4 GB minikube):** steps 2 to 6 fit.
- **Larger cluster (6 GB or more):** add Argo CD for steps 7 and 8.

## Repository layout

```
docs/          the guides above
policies/      Kyverno policies, applied by Argo CD
├── validate/
├── mutate/
└── generate/
examples/      policies that are not applied automatically (exception, CEL policy)
tests/         Kyverno CLI tests and sample resources
argocd/        root Application and the Applications for Kyverno and the policies
```

## Prerequisites
`kubectl`, `helm`, a Kubernetes cluster (minikube is fine), and optionally the `kyverno` and `argocd` CLIs.

```bash
brew install kyverno argocd
unset DOCKER_DEFAULT_PLATFORM      # an amd64 default breaks the minikube node image on Apple Silicon
minikube start --cpus=4 --memory=4096
kubectl config current-context     # must be: minikube
```
Never run these labs against a shared or production cluster. Kyverno policies in Enforce mode can block deployments.

## Credits
Based on the ideas in [iam-veeramalla/k8s-kyverno-argocd](https://github.com/iam-veeramalla/k8s-kyverno-argocd) and the [kyverno/policies](https://github.com/kyverno/policies) library (Apache-2.0). The policies and guides here were written fresh, and the CEL example follows the sample in the library README.
