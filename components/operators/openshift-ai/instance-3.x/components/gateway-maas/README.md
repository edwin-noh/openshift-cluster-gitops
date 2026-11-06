# gateway-maas

`maas-default-gateway` (openshift-ingress) for RHOAI Models-as-a-Service, reachable at
`https://maas.apps.<cluster-domain>` **without a LoadBalancer / MetalLB**:

```
client --TLS(SNI maas.apps.<domain>)--> OpenShift router (passthrough Route maas-gateway-route)
       --> Service maas-default-gateway-data-science-gateway-class (ClusterIP, via maas-gw-options)
       --> Envoy terminates TLS with router-certs-default (*.apps wildcard)
```

- `route-hostname-job` (Argo CD PostSync hook) derives `<cluster-domain>` from the console URL
  and sets it on the Route `spec.host` and every Gateway listener `hostname`. openshift-ingress is
  default-deny on OCP 4.22+ (`openshift-ingress-deny-all`), so `networkpolicy.yaml` opens DNS + API
  egress for the Job pod only.
- `router-certs-default` is signed by the ingress operator CA, so clients must trust that CA
  (or use `-k` when testing), like the console.

Ported from ai-accelerator; only the TLS secret differs.
