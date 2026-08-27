# Architecture and design choices

## Why folders instead of branches

A common alternative design gives each environment its own long-lived Git
branch (`env/platform-sandbox`, `env/app-dev`, `env/app-prod`) and promotes
a change by merging one branch into the next. This repo deliberately avoids
that. Long-lived branches drift apart over time as each accumulates its own
small changes, and merging them eventually produces conflicts that have
nothing to do with the actual change being promoted — you end up resolving
git history instead of reviewing a deploy.

Keeping everything on `main` and separating environments by folder means
`git log` on this repo is always the true, complete history of every
environment at once. A promotion is a small diff between two files that
happen to live in different folders, reviewed like any other pull request.

## Why tenants, not a flat manifest list per environment

Each team's (or the platform team's own) resources live in their own
subfolder with their own `kustomization.yaml`, rather than one big list of
resources per environment. This is a multi-tenancy pattern: it means
onboarding a new team is additive — one new folder, one new line in the
tenant list — and never requires editing another tenant's files. It also
means a mistake in one tenant's folder can't accidentally break another
tenant's Kustomize build, since each tenant's `kustomization.yaml` is only
ever combined with the others at the top level, never merged field-by-field.

`platform-services` — the platform team's own workloads — is deliberately
onboarded through this exact same tenant mechanism rather than being
special-cased. If this repo's structure has a rough edge for a real tenant,
the platform team hits it too, before any other team does.

## Why this repo has no CI/CD pipeline

Unlike `platform-team-administration` and `platform-core`, this repo has no
`.circleci/config.yml` at all — there's no pipeline that runs on push,
lints anything, or gates a merge with an automated check. This is a
deliberate difference, not an oversight:

- Every file in this repo is either static YAML or a small Kustomize
  overlay. There's no code to lint or test in the sense the other repos
  have — `kubectl kustomize` either renders successfully or fails loudly
  with a YAML/reference error (see [Troubleshooting](../how-to/troubleshooting.md)),
  and that check takes seconds to run locally before ever opening a PR.
- Argo CD itself is the continuous verification loop. Once a change merges
  to `main`, Argo CD polls this repo, renders it the same way `kubectl
  kustomize` does locally, and reports `Synced`/`Healthy` or a concrete
  sync error — there's no separate "deploy" step this repo needs to trigger.
- The actual safety gate is the pull request review, not an automated
  pipeline. This is a real, load-bearing difference from `platform-core`,
  where an approval gate is enforced by CircleCI's workflow structure
  itself. Here, that discipline depends entirely on whoever reviews the PR
  checking the previous environment's health first — see
  [Promote a change across environments](../how-to/promote-a-change-across-environments.md)
  for why that matters.

## Why the Application object lives here, not in the target repo

Each tenant's `Application` object (naming its own repo, branch, and path)
lives in `platform-gitops`, not in the tenant's own repo. This keeps one
property true throughout the whole project: this repo is the single place
Argo CD is ever pointed at, and everything else — which repos exist, which
environments they deploy to — is discoverable by reading files here, rather
than by knowing to go looking in N different repos.
