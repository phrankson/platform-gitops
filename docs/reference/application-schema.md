# Argo `Application` object reference

Full field reference for the `Application` objects this repo defines, using
`environments/platform-sandbox/tenants/platform-services/platform-services.yaml`
as the worked example. All three environments use the identical shape,
differing only in `spec.source.path` and the inherited `env` label.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-services
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/phrankson/platform-services.git
    targetRevision: main
    path: environments/platform-sandbox
  destination:
    server: https://kubernetes.default.svc
    namespace: platform-services
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
```

| Field | Value here | Effect |
|---|---|---|
| `metadata.name` | `platform-services` | The Application's own name, shown in `kubectl get application` and the Argo CD UI. |
| `metadata.namespace` | `argocd` | Every Application object lives in the `argocd` namespace regardless of where it deploys to. |
| `spec.source.repoURL` | the tenant's own repo | A different repo than `platform-gitops` itself — this is the pointer, not the payload. |
| `spec.source.targetRevision` | `main` | The branch Argo CD tracks in the target repo. |
| `spec.source.path` | `environments/<env>` (evaluated inside the *target* repo) | Which folder in the target repo to render. This is why `platform-services`'s own repo also needs its own `environments/<env>/` folders — a naming convention this project keeps consistent across repos, not something Argo CD enforces. |
| `spec.destination.server` | `https://kubernetes.default.svc` | Deploy to the same cluster Argo CD itself runs on — always true in this project, since every environment is a separate cluster with its own Argo CD instance, not one central Argo CD managing many remote clusters. |
| `spec.destination.namespace` | `platform-services` | Namespace the rendered resources land in. |
| `spec.syncPolicy.automated.prune` | `true` | Delete cluster resources that are no longer in the source repo. |
| `spec.syncPolicy.automated.selfHeal` | `true` | Revert manual cluster changes back to match the source repo automatically. |
| `spec.syncPolicy.syncOptions` | `[CreateNamespace=true]` | Create `spec.destination.namespace` if it doesn't already exist. |
| `spec.syncPolicy.retry.limit` | `5` | Maximum retry attempts on a failed sync. |
| `spec.syncPolicy.retry.backoff` | `5s` initial, `3m` max | Exponential backoff between retries. |

## The root Application (not defined in this repo)

The first `Application` in the whole chain — the one that points at this
repo — is created by `platform-core`'s `seed_gitops()`, not by any file
here. See the `seed_gitops` entry in
[`platform-core`'s Modules API reference](../../../platform-core/docs/reference/modules-api.md)
for that object's fields.
