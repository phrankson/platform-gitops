# Onboard a new tenant

Use this to add a new team's (or the platform team's own) workloads to an
environment, without touching any existing tenant's setup.

## Steps

1. Create a folder for the tenant under that environment's `tenants/`
   directory. For a team called `checkout` in `platform-sandbox`:

   ```console
   $ mkdir -p environments/platform-sandbox/tenants/checkout
   ```

2. Add that tenant's own `kustomization.yaml`, listing whatever manifests or
   Argo `Application` objects the tenant wants deployed. Following the
   existing `platform-services` tenant's pattern:

   ```yaml
   # environments/platform-sandbox/tenants/checkout/kustomization.yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization

   resources:
     - checkout.yaml
   ```

3. Add the actual `Application` object (or plain manifests) the
   kustomization above references — for example
   `environments/platform-sandbox/tenants/checkout/checkout.yaml`, pointing
   at the team's own repo.

4. Register the tenant by adding one line to the environment's tenant list:

   ```yaml
   # environments/platform-sandbox/tenants/kustomization.yaml
   resources:
     - platform-services
     - checkout
   ```

5. Verify the chain renders correctly before opening a PR:

   ```console
   $ kubectl kustomize environments/platform-sandbox
   ```

   You should see both tenants' objects in the output.

6. Open a normal pull request against `main`. Once merged, Argo CD picks up
   the change on its next reconciliation pass — no manual `kubectl apply`,
   no restart, nothing else to trigger.

## What you don't need to touch

Nothing about `platform-services`'s folder, or any other existing tenant's
folder, changes. The only shared file you edit is the one tenant list in
step 4 — every other file involved in this repo's chain (the environment's
root `kustomization.yaml`) stays exactly as it was.

## Repeating this for other environments

This process is per-environment. Onboarding `checkout` to `platform-sandbox`
does nothing for `app-dev` or `app-prod` — repeat steps 1–5 under each
environment folder you want the tenant present in. See
[Add a new environment](add-a-new-environment.md) if the environment itself
doesn't exist yet.
