# components-modelregistry

## Purpose

This component enables the Model Registry in the DataScienceCluster. Model Registry provides a central repository to store, version, and manage machine learning models and their metadata.

Models are stored in the `rhoai-model-registries` namespace by default.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-modelregistry
```
