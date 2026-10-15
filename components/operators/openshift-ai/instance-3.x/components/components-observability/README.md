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
