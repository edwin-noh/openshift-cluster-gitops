# cert-manager Operator for Red Hat OpenShift

Installs the Red Hat build of cert-manager. The operator itself runs in
`cert-manager-operator`; the cert-manager controller, webhook and CA injector it
deploys land in the `cert-manager` namespace.

Required by:

- `components/cluster-configs/self-signed-internal-ca` — creates `ClusterIssuer`
  `selfsigned-clusterissuer` / `ca-issuer` and a `Certificate` in the `cert-manager`
  namespace.
- RHOAI 3.5 itself — the docs list cert-manager as a prerequisite for `kserve`,
  `kueue` / `ray` / `trainingoperator`, and `ogx`.
- any other component issuing a `cert-manager.io/v1 Certificate`.

Upstream `ai-accelerator` does not list this operator because RHDP sandbox clusters
ship cert-manager pre-installed. On a self-managed cluster it has to be installed
explicitly, otherwise `self-signed-internal-ca` never syncs.
