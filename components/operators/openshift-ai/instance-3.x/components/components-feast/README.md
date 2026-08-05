# components-feast

## Purpose

This component enables the Feast Operator in the DataScienceCluster. Feast is an open-source feature store that helps teams manage, store, and serve machine learning features for both training and inference.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-feast
```
