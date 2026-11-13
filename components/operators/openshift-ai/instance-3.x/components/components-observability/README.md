# components-observability

## Purpose

Turns on RHOAI platform observability by patching the `DSCInitialization`: metrics
(Prometheus) into `redhat-ods-monitoring` and traces (Tempo) onto a persistent volume.
The base leaves `monitoring.managementState: Removed`, because the docs require
`metrics.storage.size` and `.retention` whenever monitoring is `Managed` and the base
has no sensible value for them — this component supplies both.

Requires the Cluster Observability Operator, Tempo and the Red Hat build of
OpenTelemetry to be installed first.

## Sizing caution

The defaults here (`metrics.storage.size: 50Gi` / `retention: 90d`, traces
`retention: 2160h`) come from a multi-node reference cluster; the RHOAI docs use 5Gi in
their example. On a single node backed by NFS this is worth shrinking:

- Running a Prometheus TSDB on NFS is not supported by Red Hat — file locking and mmap
  behaviour over NFS can corrupt the database. Tempo with `backend: pv` has the same
  exposure through its WAL.
- The nfs-subdir provisioner enforces no quota, so `50Gi` is a nominal number; it will
  happily fill the export.

## Viewing traces

Traces land in the `data-science-tempomonolithic` TempoMonolithic in
`redhat-ods-monitoring` (apps send OTLP to `data-science-collector` on 4317/4318).
This component also adds:

- `UIPlugin distributed-tracing` (Cluster Observability Operator): enables the console
  **Observe -> Traces** page. Pick `redhat-ods-monitoring / data-science-tempomonolithic`
  and tenant `redhat-ods-monitoring`.
- `traces-rbac.yaml`: the Tempo runs in `openshift` multitenancy mode, where the gateway
  checks `create` (write) / `get` (read) on `tempo.grafana.com/redhat-ods-monitoring`
  (resourceName `traces`). RHOAI 3.5 grants neither, so without this file the collector's
  `otlp/tempo` exporter fails with `PermissionDenied: forbidden` and drops every span.
  - `rhoai-traces-writer` -> SA `redhat-ods-monitoring/data-science-collector-collector`
  - `rhoai-traces-reader` -> groups `rhods-admins`, `kbl-ai-ops` (cluster-admin can
    always read). Add groups here to widen access.

## Dashboards (Perses)

`UIPlugin monitoring` with `perses.enabled: true` (COO 1.5+, GA) adds the console
**Observe -> Dashboards (Perses)** page. COO runs a separate Perses for the console; the
RHOAI-deployed `data-science-perses` in `redhat-ods-monitoring` keeps serving the RHOAI
dashboard tabs. Custom dashboards are `PersesDashboard` + `PersesDatasource` CRs in a
project (see `kblife-rhoai/observability`).

Kiali is not needed: it is a Service Mesh console, and RHOAI 3.x has no Service Mesh.
The RHOAI docs only require COO + Tempo + OpenTelemetry and suggest Grafana or Jaeger
as external viewers.

## Usage

This component can be added to a base by adding the `components` section to your
overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-observability
```
