# Troubleshooting

Real, reproducible failure modes in this repo's Kustomize chain, and what
Argo CD does (or doesn't do) when it hits each one.

## A new tenant folder never gets deployed, with no error anywhere

**Symptom:** you added `environments/<env>/tenants/<team>/`, it has a valid
`kustomization.yaml`, `git log` shows it merged — but `kubectl get
application` never shows anything for it, and there's nothing in Argo CD's
UI or logs about it either.

**Cause:** you added the folder but forgot to add it to
`environments/<env>/tenants/kustomization.yaml`. This is the single most
important thing to check, because it fails silently — verified directly:

```console
$ kubectl kustomize environments/platform-sandbox
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  labels:
    env: platform-sandbox
  name: platform-services
...
```

A second tenant folder existing on disk, unlisted, produces exactly the
same output as if it didn't exist — no warning, no error, exit code 0. Both
Kustomize and Argo CD treat "not listed" and "not present" identically.

**Fix:** add the tenant's folder name to the `resources:` list in
`tenants/kustomization.yaml` for that environment, then re-render to
confirm the tenant now appears in the output.

## `kubectl kustomize` fails with "must resolve to a file"

**Symptom:**

```console
$ kubectl kustomize environments/platform-sandbox
error: accumulating resources: accumulation err='accumulating resources
from 'tenants': ... accumulation err='accumulating resources from
'team-checkout': evalsymlink failure on
'.../tenants/team-checkout': lstat .../tenants/team-checkout: no such file
or directory'
```

**Cause:** the opposite of the case above — `tenants/kustomization.yaml`
lists a tenant folder that doesn't exist (a typo in the name, or the folder
was never created, or was renamed without updating this file).

**Fix:** check that the name listed under `resources:` exactly matches an
existing folder name under `tenants/`. Case and hyphenation have to match
exactly — Kustomize does not fuzzy-match folder names.

## `kubectl kustomize` fails with a YAML parse error

**Symptom:**

```console
$ kubectl kustomize environments/platform-sandbox
error: accumulating resources: ... couldn't make target for path
'.../tenants': invalid Kustomization: yaml: line 1: did not find expected
node content
```

**Cause:** a `kustomization.yaml` somewhere in the chain has invalid YAML —
often a stray bracket, bad indentation, or a half-finished edit committed by
mistake.

**Fix:** run `kubectl kustomize` locally against the affected environment
before opening a PR (see [Getting started](../tutorials/getting-started.md))
— this always catches YAML errors immediately, since Kustomize has to fully
parse every file in the chain to render anything at all.

## Argo CD shows the Application as `OutOfSync` even though nothing changed

**Cause:** this is a real, currently-observable state on this project's
sandbox cluster — some `Application` objects report `OutOfSync` while still
showing `Healthy`, meaning the live resources are working correctly but
don't byte-for-byte match the last-applied manifest (commonly caused by a
controller, like Istio's own sidecar injector, mutating a resource after
Argo CD applies it). This is not this repo's problem to fix — it's a
property of whatever's being deployed. If you see this, check that specific
tenant's own repo (for example, `platform-services`'s
[troubleshooting notes](../../../platform-services/learning/service-mesh.md))
before assuming something in `platform-gitops` is wrong.
