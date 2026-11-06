# components-aigateway-maas

Turns on RHOAI 3.5 Models-as-a-Service:

- DSC `aigateway.managementState: Managed` + `aigateway.modelsAsAService.managementState: Managed`
  (3.5 path; the older `kserve.modelsAsService` is a legacy field)
- `redhat-ai-gateway-infra` namespace (MaaS infra namespace, allowed on `maas-default-gateway`)
- `MaasTenantConfig default-tenant` (telemetry on)
- Dashboard flags: `modelAsService`, `maasAuthPolicies`, `externalModels`

Use together with:

- `gateway-maas`: `maas-default-gateway` plus the Route
- `maas-postgres`: API key store and the `maas-db-config` secret
- the `redhat-connectivity-link` operator (`aggregate/overlays/maas`), which provides Kuadrant, Authorino and Limitador
- User Workload Monitoring (`cluster-configs/user-workload-monitoring`)

Namespaces that hold models served through MaaS need the label
`maas.opendatahub.io/gateway-access: "true"` so their HTTPRoutes can attach to the gateway.
