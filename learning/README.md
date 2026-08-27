# Learning: platform-gitops

A teaching companion for this repo, continuing from
[`platform-core`'s learning companion](../../platform-core/learning/README.md).
That repo installs a smart-home hub in each house and pairs it to an
account. This repo is that account: not a house, not a builder, just a
set of instructions the hub checks continuously and acts on by itself.

1. **[The App of Apps Pattern](app-of-apps.md)** — the actual chain of
   files Argo CD reads, field by field, and how a new team gets added
   without touching anyone else's setup.

---

## One filing cabinet, organized by house, then by resident

This repo holds no application code. It's closer to a filing cabinet: a
folder for each house (`platform-sandbox`, `app-dev`, `app-prod`), and
inside each house's folder, one sub-folder per resident who has something
running there. Argo CD, once paired to this repo, keeps checking that
cabinet and making the real cluster match whatever's filed in it.

```
environments/
  platform-sandbox/
    kustomization.yaml
    tenants/
      kustomization.yaml
      platform-services/
        kustomization.yaml
        platform-services.yaml
  app-dev/
    ...same shape...
  app-prod/
    ...same shape...
```

Each resident's folder is called a **tenant**. Right now there's exactly
one, `platform-services`, but the shape is built to hold more without
changing anything that's already there.

## Why one branch, not one branch per house

A common mistake in setups like this is giving each house its own
long-lived Git branch — `env/platform-sandbox`, `env/app-dev`,
`env/app-prod` — and promoting a change by merging one branch into the
next. That sounds reasonable until the branches drift apart over time,
and merging them starts producing conflicts that have nothing to do with
the actual change being promoted.

This repo only has one branch, `main`. Environments are separated by
folder instead. Promoting a change means editing a file under the target
environment's folder and opening a normal pull request against `main` —
never merging one branch into another. Every environment's current state
is always visible in the same branch, at the same time, which is also
why `git log` on this repo tells you the true history of every
environment at once, instead of three separate, diverging stories.

## Multi-tenancy: one platform, many residents, no interference

The word tenant is doing real work here, borrowed directly from
multi-tenancy — the pattern of running many independent customers or
teams on one shared piece of infrastructure, isolated enough from each
other that nobody notices anyone else is even there.

Look at what onboarding a second tenant would actually require. You'd add
a folder — say, `environments/platform-sandbox/tenants/team-checkout/` —
with its own `kustomization.yaml` describing that team's own resources,
and add one line to `environments/platform-sandbox/tenants/kustomization.yaml`
pointing at it. That's the entire onboarding process. Nothing about
`platform-services`'s existing folder changes. Nothing about how
`platform-services` gets deployed changes. The two tenants share the same
environment, the same Argo CD instance, the same underlying cluster, and
never have to know the other one exists.

This is also why `platform-services` — the platform team's own workloads —
lives here as a tenant like any other, instead of being handled as a
special case. If the platform team wants proof this pattern actually
works before asking another team to trust it, running their own stuff
through the identical path is the only honest way to get that proof.

Continue to [**The App of Apps Pattern**](app-of-apps.md) for what's
actually inside each of those `kustomization.yaml` files, and what Argo CD
does with them.
