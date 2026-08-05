# idle-notebook-culling

## Purpose

This component configures automatic culling of idle notebook servers. When enabled, notebooks that remain idle for a specified period will be automatically stopped to free up cluster resources.

Default configuration:
- **ENABLE_CULLING**: `true` - Culling is enabled
- **CULL_IDLE_TIME**: `240` minutes (4 hours) - Time before an idle notebook is stopped
- **IDLENESS_CHECK_PERIOD**: `1` minute - How often to check for idle notebooks

For more information, see the official docs for [Stopping Idle Notebooks](https://access.redhat.com/documentation/en-us/red_hat_openshift_ai_self-managed/2-latest/html/managing_resources/managing-notebook-servers_notebook-mgmt#stopping-idle-notebooks_notebook-mgmt).

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/idle-notebook-culling
```

You can customize the idle timeout settings by modifying the [culler-config.yaml](./culler-config.yaml) file.
