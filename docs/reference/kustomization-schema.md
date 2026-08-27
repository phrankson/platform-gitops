# Kustomization file reference

Three distinct kinds of `kustomization.yaml` exist in this repo, one per
level of the chain. All three share the same `apiVersion`/`kind`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```

## Environment root — `environments/<env>/kustomization.yaml`

| Field | Value in this repo | Effect |
|---|---|---|
| `labels[].pairs` | `{env: <env-name>}` | Stamps every rendered resource in this environment with a matching label. |
| `labels[].includeSelectors` | `true` | Also applies the label to any `matchLabels`/`selector` fields on rendered resources, not just their own metadata. |
| `labels[].includeTemplates` | `true` | Also applies the label inside any embedded pod templates. |
| `resources` | `[tenants]` | Pulls in the tenant list one level down. |

Current instances: `platform-sandbox` (`env: platform-sandbox`), `app-dev`
(`env: app-dev`), `app-prod` (`env: app-prod`) — identical shape, differing
only in this label.

## Tenant list — `environments/<env>/tenants/kustomization.yaml`

| Field | Value | Effect |
|---|---|---|
| `resources` | list of tenant folder names | Every listed folder must exist under `tenants/` and contain its own valid `kustomization.yaml`. A folder that exists but isn't listed here is silently excluded — see [Troubleshooting](../how-to/troubleshooting.md). |

Current instances: all three environments currently list exactly one
tenant, `platform-services`.

## Tenant's own kustomization — `environments/<env>/tenants/<tenant>/kustomization.yaml`

| Field | Value | Effect |
|---|---|---|
| `resources` | list of filenames | Whatever that tenant wants deployed — typically one or more Argo `Application` objects (see [Application reference](application-schema.md)), but plain Kubernetes manifests are equally valid here. |

Current instance: `platform-services/kustomization.yaml` lists exactly one
file, `platform-services.yaml`.
