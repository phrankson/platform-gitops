# Promote a change across environments

Use this to move a change — a new image tag, a config value, a new tenant —
from `platform-sandbox` through `app-dev` to `app-prod`.

## The pattern

There is no environment branch to merge and no promotion pipeline in this
repo. Promoting a change means editing the same file under a different
environment folder and opening a separate pull request for each hop:

```console
$ diff environments/platform-sandbox/tenants/platform-services/platform-services.yaml \
       environments/app-dev/tenants/platform-services/platform-services.yaml
```

The two files differ only in `spec.source.path` (`environments/platform-sandbox`
vs. `environments/app-dev`, evaluated inside the *target* repo, not this one)
and the `env` label each environment's root kustomization stamps on. Everything
else is identical by design — a promotion should be a small, reviewable diff,
not a rewrite.

## Steps

1. Make the change under `environments/platform-sandbox/...` first, and get
   it merged and confirmed healthy there:

   ```console
   $ kubectl get application platform-services -n argocd
   NAME                SYNC STATUS   HEALTH STATUS
   platform-services   Synced        Healthy
   ```

2. Copy the same change into the equivalent file under
   `environments/app-dev/...`. Open a pull request scoped to only that
   environment's folder — resist the urge to bundle the sandbox and app-dev
   changes into one PR, since a reviewer should be able to see exactly which
   environment is being touched.

3. Merge, confirm `app-dev` is healthy the same way as step 1, then repeat
   for `environments/app-prod/...`.

## Why this is a manual, per-environment PR chain

This repo has no CI/CD pipeline (see
[why this repo has no pipeline](../explanation/architecture-and-design-choices.md#why-this-repo-has-no-cicd-pipeline)).
The pull request itself — one reviewer approving one small diff — is the
gate at every hop. Compare this to `platform-core`, where the equivalent
promotion is enforced by CircleCI's approval-gated workflow; here it's
enforced by whoever reviews the PR remembering to check the previous
environment's health first. That's a real gap worth knowing about, not a
hidden safety net.
