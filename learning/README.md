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

## First, what does "the hub checks continuously" actually mean?

This is worth slowing down on, because it's the single most important idea
in how this whole project works, and it's easy to read past it without
really absorbing it.

Picture a contractor who renovates a house once, checks their work, and
leaves. That's how most deployments work without a tool like Argo CD:
someone (or some CI pipeline) runs a command, it makes some changes, and
then nobody is watching anymore. If a different contractor comes by later
and swaps out a fixture, or something breaks on its own, nobody notices
until a resident happens to complain.

Argo CD is not that contractor. It's more like a live-in facilities person
who was handed a copy of the house's blueprint (this repo) on day one, and
who never stops walking the halls comparing what they see to what the
blueprint says. Every few minutes, forever, for as long as the hub is
running: walk through, compare, fix anything that doesn't match, walk
through again. There is no "finished." The checking never stops.

This explains a few things that would otherwise look strange:

- If someone manually changes something in the house that this account's
  instructions describe — swaps a fixture by hand, say — the hub quietly
  puts it back, usually within a few minutes, without anyone asking it to.
  This isn't a bug or the hub being stubborn; it's the whole point of
  having a live-in facilities person instead of a one-time contractor.
- "Sync status" is the answer to "does the house currently match the
  blueprint?" "Health status" is a separate question: "is the house
  actually livable right now, whether or not it matches the blueprint
  exactly?" A house can be perfectly livable while still not matching the
  blueprint down to the last detail — see the real, currently-running
  example of exactly that in
  [`platform-services`'s companion](../../platform-services/learning/service-mesh.md),
  where two of the three Istio pieces show `OutOfSync` and `Healthy` at
  the same time.
- Nothing in this repo ever gets "deployed" in the sense of a single
  command running once. A change here just edits the blueprint. The hub
  finds the edit on its own next walkthrough and makes the house match it.

## Then, which "environment" are we even talking about?

`platform-core`'s companion already used the word "house" for one specific
thing: an entire separate structure, built from scratch, with its own
foundation and wiring — never a room inside a shared building. That's
worth holding onto, because this repo reuses the same underlying idea
under a different name, and then reuses it again a third time, and all
three are correct simultaneously:

- To `platform-core`, an environment is a whole separate house — a Kind
  cluster, with nothing shared between it and any other house.
- To this repo, an environment is a drawer in the filing cabinet — a
  folder under `environments/`, holding instructions for exactly one
  house and no other.
- To anything Argo CD actually produces from those instructions, an
  environment is a tag stapled to it — `env: platform-sandbox`, say — so
  that if you ever looked at a pile of instructions from every house mixed
  together, you could still tell at a glance which house each one belongs
  to.

None of these three is the "real" definition, with the others as loose
shorthand for it. Each one is exactly the right tool for what its own
layer needs: a builder needs a real, physical house to build; a filing
system needs a drawer to organize paperwork by; a label needs to be small
enough to staple to a single sheet. The habit worth building, every time
you read "environment" anywhere in this project's docs, is asking which of
the three is actually meant — the house, the drawer, or the tag. Once
that's automatic, nothing about this repo's folder layout will feel
inconsistent.

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
