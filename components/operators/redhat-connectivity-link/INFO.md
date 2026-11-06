# redhat-connectivity-link

Red Hat Connectivity Link (upstream: Kuadrant). It provides the Authorino (authn/authz) and
Limitador (rate limiting) data-plane policies that RHOAI Models-as-a-Service compiles its
MaaSAuthPolicy / MaaSSubscription objects into. RHOAI 3.5 needs Kuadrant v1.4.2+.

Ported from ai-accelerator `components/operators/redhat-connectivity-link`, keeping only
what `aggregate/overlays/maas` uses. Differences:

- No `ConsolePlugin` in git. The operator creates and owns `kuadrant-console-plugin`, so a
  copy in git would stay OutOfSync and selfHeal would loop. The Job only enables it.
- No blanket `commonAnnotations` in the aggregate. That would overwrite the per-object
  `ServerSideApply=true` / `Force=true,Replace=true` sync options, so each object sets its own.
