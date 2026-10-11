# cert-manager Operator for Red Hat OpenShift

Installs the Red Hat build of cert-manager. The operator itself runs in
`cert-manager-operator`; the cert-manager controller, webhook and CA injector it
deploys land in the `cert-manager` namespace.

Required by:

- `components/cluster-configs/self-signed-internal-ca` — creates `ClusterIssuer`
  `selfsigned-clusterissuer` / `ca-issuer` and a `Certificate` in the `cert-manager`
  namespace.
- any component issuing a `cert-manager.io/v1 Certificate` (e.g. the MaaS CNPG
  Postgres server cert).

Upstream `ai-accelerator` does not list this operator because RHDP sandbox clusters
ship cert-manager pre-installed. On a self-managed cluster it has to be installed
explicitly, otherwise `self-signed-internal-ca` never syncs.
