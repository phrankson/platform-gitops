# platform-gitops

The single coordination hub Argo CD watches. This repo holds no application
code — just declarative pointers to what should be deployed where.

## Why one branch, not one branch per environment

Environments are separated by **folder**, not by long-lived Git branch.
There is only ever `main`. Promoting a change means editing a file under the
target environment's folder and opening a normal PR — never merging one
environment branch into another.

## Layout

```
environments/
  platform-sandbox/
    kustomization.yaml       # lists which tenants apply here
    tenants/
      platform-team/         # dogfooding: platform team is tenant #1
  app-dev/
    ...same shape...
  app-prod/
    ...same shape...
```

Each environment folder matches a `platform-core` Pulumi stack of the same
name. Each tenant folder is a self-contained Kustomize target — a team's
(or the platform team's own) manifests for that one environment.

This structure is currently hand-maintained. That's a deliberate trade-off
for a small number of environments/tenants — worth revisiting (templates or
a generator) if that count grows enough to make hand-maintenance a burden.
