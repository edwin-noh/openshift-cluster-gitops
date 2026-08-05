# components-llamastack

## Purpose

This component enables the LlamaStack Operator in the DataScienceCluster. LlamaStack provides tooling and infrastructure for deploying and managing Llama models.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-llamastack
```
