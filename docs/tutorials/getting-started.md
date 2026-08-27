# Getting started

This tutorial takes you from a fresh clone to rendering this repo's full
manifest chain locally, and comparing it against what's actually running on
a real cluster. No changes, no risk — just seeing what this repo produces.

## Prerequisites

- [kubectl](https://kubernetes.io/docs/tasks/tools/) (used here only for its
  `kustomize` subcommand — no cluster access required for this step)
- Access to a cluster running Argo CD, if you want to complete step 3

## Step 1: Clone the repository

```console
$ git clone https://github.com/phrankson/platform-gitops.git
$ cd platform-gitops
```

## Step 2: Render an environment's full manifest chain

Every environment folder is a Kustomize root. Rendering it resolves the
entire chain — root kustomization, tenant list, each tenant's own
kustomization — down to the actual objects Argo CD will apply:

```console
$ kubectl kustomize environments/platform-sandbox
```

You should see one Argo `Application` object, named `platform-services`,
labeled `env: platform-sandbox`. Nothing was deployed — `kubectl kustomize`
only renders YAML to stdout, it never talks to a cluster.

Try the other two environments and compare:

```console
$ kubectl kustomize environments/app-dev
$ kubectl kustomize environments/app-prod
```

The output is nearly identical for all three — same tenant, same object
structure — except for the `env` label and the `spec.source.path`, which
each point at that environment's own path in the `platform-services` repo.
That's the whole point of the folder-per-environment structure: one pattern,
repeated, not three different setups to reason about.

## Step 3: Compare against the live cluster

If you have `kubectl` access to a cluster where this repo is already paired
to Argo CD:

```console
$ kubectl get application -n argocd
```

You should see a `platform-services` Application with `SYNC STATUS: Synced`
and `HEALTH STATUS: Healthy` — meaning the object you rendered in step 2 is
already applied, and Argo CD has confirmed the real cluster matches it.

## What you've done

You've rendered this repo's actual deploy instructions without touching
Argo CD, and confirmed what "synced" means: the file on disk and the object
in the cluster agree. From here:

- To add a new team's workloads, see
  [Onboard a new tenant](../how-to/onboard-a-new-tenant.md).
- To understand every field in the files you just rendered, see the
  [Kustomization reference](../reference/kustomization-schema.md) and the
  [Application reference](../reference/application-schema.md).
