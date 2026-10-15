# MetalLB Operator

Provides `type: LoadBalancer` Services on bare metal / KVM, where there is no cloud
load balancer.

Required because RHOAI 3.5 exposes the dashboard (`rh-ai.apps.<cluster-domain>`) and
inference endpoints through Gateway API. When a Gateway is accepted, the ingress
controller creates a `LoadBalancer` Service for it; without MetalLB the external IP
stays `<pending>` and `Gateway.status.conditions[Programmed]` never goes true.

The RHOAI install docs call this out for "OpenStack, CodeReady Containers (CRC), and
other private cloud environments without integrated external DNS" — a libvirt SNO is
exactly that case.

Layout: `operator/overlays/stable` (Subscription) + `instance/overlays/default`
(`MetalLB` CR, `IPAddressPool`, `L2Advertisement`), bundled by
`aggregate/overlays/default`.

## Environment-specific

`instance/base/ipaddresspool.yaml` hardcodes `192.168.31.240-192.168.31.250`. The range
must be free on the node's L2 segment and outside the DHCP pool. Check before applying:

```bash
nmap -sn 192.168.31.240-250
```

## After install

MetalLB only assigns the IP. DNS is still manual — point a wildcard
`*.apps.<cluster-domain>` (or at least `rh-ai.apps.<cluster-domain>`) A record at the
address MetalLB hands the RHOAI gateway:

```bash
oc get svc -n openshift-ingress -o wide | grep LoadBalancer
```
