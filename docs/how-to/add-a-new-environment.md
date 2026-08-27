# Add a new environment

Use this when `platform-core` provisions a new cluster/stack and it needs a
matching environment folder here.

## Prerequisite

The environment must already exist as a `platform-core` Pulumi stack, and
that stack's `seed_gitops()` call must be pointed at the path you're about to
create — `platform-core` seeds the root Argo `Application` using
`path=f"environments/{pulumi.get_stack()}"`, so the folder name here has to
match the stack name exactly.

## Steps

1. Create the environment folder and its root kustomization, copying an
   existing one and changing only the `env` label:

   ```console
   $ mkdir -p environments/<new-env>/tenants
   ```

   ```yaml
   # environments/<new-env>/kustomization.yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization

   labels:
     - pairs:
         env: <new-env>
       includeSelectors: true
       includeTemplates: true

   resources:
     - tenants
   ```

2. Create an empty tenant list:

   ```yaml
   # environments/<new-env>/tenants/kustomization.yaml
   resources: []
   ```

3. Verify it renders (an empty tenant list is valid — it just produces no
   output):

   ```console
   $ kubectl kustomize environments/<new-env>
   ```

4. Onboard whichever tenants should exist in this environment — see
   [Onboard a new tenant](onboard-a-new-tenant.md).

5. Open a PR. Once merged and once `platform-core` has provisioned the
   cluster and paired Argo CD to this repo at `environments/<new-env>`, Argo
   CD will pick up whatever tenants you added in step 4.

## Verifying the pairing from the other side

If you have access to the new cluster, confirm Argo CD is actually watching
the path you created:

```console
$ kubectl get application platform-gitops -n argocd -o jsonpath='{.spec.source.path}'
```

This should print `environments/<new-env>`. If it prints something else, the
mismatch is on the `platform-core` side — see that repo's
[troubleshooting guide](../../../platform-core/docs/how-to/troubleshooting.md).
