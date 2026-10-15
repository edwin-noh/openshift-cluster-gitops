# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this repo is

**Kustomize manifests** for declarative OpenShift Day2 configuration, applied by in-cluster Argo CD.
Ansible automation (KVM/SNO install, GitOps bootstrap) lives in a separate repo (`~/Workspace/openshift-ansible`).

## GitOps pipeline

Ansible (separate repo) installs the GitOps Operator and creates a `bootstrap` Application.
**Argo CD** then pulls Kustomize manifests from this repo to apply Day2 configuration.

**Important:** Argo CD pulls from the **remote git**, so manifest changes must be committed + pushed.
Local `kustomize build` passing ≠ cluster updated.

## Kustomize layering (how a cluster type is assembled)

A "cluster type/version" is selected by pointing the Argo CD bootstrap Application at a
`bootstrap/overlays/<cluster>` directory. The chain for each `<cluster>` (currently only `rhoai-3.5`):

```
bootstrap/overlays/<cluster>            → resources: clusters/overlays/<cluster>
clusters/overlays/<cluster>             → namespace: openshift-gitops
                                          resources: ../../base (AppProjects)
                                                   + components/argocd/apps/overlays/<cluster>
                                          replacements: (see below)
components/argocd/apps/base              → Application "cluster-config-app-of-apps"
                                          + ApplicationSet "cluster-configs"
                                          + ApplicationSet "cluster-operators"
components/argocd/apps/overlays/<cluster> → patches the three objects above with concrete values
```

The actual operators/configs are delivered by the two **ApplicationSets**, whose `list` generators
(set in each overlay's `patch-operators-list.yaml` / `patch-configs-list.yaml`) point at
`components/operators/<op>/...` and `components/cluster-configs/<cfg>/...` paths.

### Two non-obvious mechanisms

- **repoURL is propagated by `replacements`.** Each `clusters/overlays/<cluster>/kustomization.yaml`
  has a `replacements` block that copies `spec.source.repoURL` and `targetRevision` from the
  `cluster-config-app-of-apps` **Application** into both ApplicationSet templates. So the app-of-apps
  is the single source of repo coordinates *within a build*, and it must exist for replacements to
  resolve. The repoURL itself lives hardcoded in
  `components/argocd/apps/base/cluster-config-app-of-apps.yaml` and **must equal** the Ansible
  `gitops_repo_url` (in the separate ansible repo's `vars/main.yml`). Changing repos means editing both places.

- **Intentional `patch-me-*` placeholders.** Base manifests carry placeholder strings that overlays
  are *required* to patch: operator Subscription `channel:` (patched in `operator/overlays/<channel>/`),
  ApplicationSet generator `elements:` and `source.repoURL`, and the app-of-apps `source.path`.
  **A successful `kustomize build` does not mean placeholders were replaced** — always grep the
  rendered output for `patch-me`.

### Component (operator) layout convention
`components/operators/<op>/` uses `operator/{base,components,overlays}` (subscription; channel set by
overlay patch, not base) + optional `instance/{base,components,overlays}` + `aggregate/overlays/<x>`
(bundles operator+instance, this is what ApplicationSets reference). `cluster-config/` subdirs under
`clusters/overlays/<cluster>/` are the app-of-apps extension point (empty placeholders by default).

## Commands

Tooling present: `kustomize` v5.x, `oc`. No `kubectl`.

```bash
# Validate ALL overlays build — the primary local test for any manifest change
for p in bootstrap/overlays/rhoai-3.5 \
         clusters/overlays/rhoai-3.5 \
         components/argocd/apps/overlays/rhoai-3.5 \
         components/operators/openshift-ai/aggregate/overlays/eus-3.5-nvidia-gpu \
         components/operators/metallb/aggregate/overlays/default; do
  kustomize build "$p" >/dev/null && echo "OK $p" || echo "FAIL $p"
done

# Verify placeholders were patched (MUST print 0) and repoURL is consistent
kustomize build bootstrap/overlays/rhoai-3.5 | grep -c patch-me
kustomize build bootstrap/overlays/rhoai-3.5 | grep -E 'repoURL:' | sort -u

# ApplicationSet list `path:` values are runtime git paths — kustomize build will NOT
# catch a typo there. Check every referenced component overlay actually exists.
kustomize build bootstrap/overlays/rhoai-3.5 \
  | grep -oE 'components/(operators|cluster-configs)/[^ ]+' | sort -u \
  | while read -r p; do [ -d "$p" ] || echo "MISSING $p"; done
```

## When adding a new cluster type/version overlay

Replicate the full chain consistently (a partial chain renders but silently does nothing):
`bootstrap/overlays/<c>` → `clusters/overlays/<c>` (must include `namespace: openshift-gitops` **and**
the `replacements` block) + `clusters/overlays/<c>/cluster-config/` placeholder →
`components/argocd/apps/overlays/<c>` patching all three base objects (app-of-apps path,
`cluster-operators` list, `cluster-configs` list). ApplicationSet list `path:` values are runtime git
paths — `kustomize build` will NOT catch a typo there, so verify the referenced component overlays exist.

