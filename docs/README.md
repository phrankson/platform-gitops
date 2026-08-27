# platform-gitops documentation

This is task and reference documentation for engineers who need to work with
this repo directly — onboard a tenant, promote a change, or debug why
something isn't deploying. It assumes general Kubernetes and GitOps
familiarity, not prior context on this project.

If you're new to platform engineering concepts and want the reasoning behind
this repo's design taught from the ground up, see the
[learning companion](../learning/README.md) instead. That's a different
document with a different audience — this one assumes you already know why
GitOps exists and just need to get something done in this specific repo.

## Structure

- **[Tutorials](tutorials/)** — learning by doing. Start here if this is
  your first time touching the repo.
- **[How-to guides](how-to/)** — task-oriented recipes: onboard a tenant,
  promote a change, add an environment, fix a broken sync.
- **[Reference](reference/)** — the exact schema of every file in this repo.
- **[Explanation](explanation/)** — the design decisions behind the repo's
  structure, and why it deliberately has no CI/CD pipeline.

## Start here

1. [Getting started](tutorials/getting-started.md)
2. [Onboard a new tenant](how-to/onboard-a-new-tenant.md)
3. [Troubleshooting](how-to/troubleshooting.md) when something doesn't sync
