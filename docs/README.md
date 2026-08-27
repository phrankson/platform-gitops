# platform-gitops documentation

This is task and reference documentation for engineers who need to work with
this repo directly — onboard a tenant, promote a change, or debug why
something isn't deploying. It doesn't assume prior context on this
project, but it does assume general Kubernetes familiarity. If Argo CD or
GitOps itself is new to you, start with [Concepts](concepts/) before
anything else.

If you're new to platform engineering concepts and want the reasoning behind
this repo's design taught from the ground up, with analogies and real
incidents, see the [learning companion](../learning/README.md) instead —
a different document, for a different purpose, but it covers a lot of the
same ground as Concepts below in a more narrative style.

## Structure

- **[Concepts](concepts/)** — the fundamentals this repo assumes you
  already know: what Argo CD actually does, and what "environment" means
  in this project specifically (it means three different concrete things
  depending which repo you're looking at). Start here if you're new to
  GitOps.
- **[Tutorials](tutorials/)** — learning by doing. Start here if you
  already know the concepts and this is your first time touching the repo.
- **[How-to guides](how-to/)** — task-oriented recipes: onboard a tenant,
  promote a change, add an environment, fix a broken sync.
- **[Reference](reference/)** — the exact schema of every file in this repo.
- **[Explanation](explanation/)** — the design decisions behind the repo's
  structure, and why it deliberately has no CI/CD pipeline.

## Start here

1. New to Argo CD or GitOps? [What is Argo CD, actually](concepts/what-is-argocd.md),
   then [What is an environment?](concepts/what-is-an-environment.md)
2. [Getting started](tutorials/getting-started.md)
3. [Onboard a new tenant](how-to/onboard-a-new-tenant.md)
4. [Troubleshooting](how-to/troubleshooting.md) when something doesn't sync
