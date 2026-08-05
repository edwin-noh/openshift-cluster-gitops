# components-ray

## Purpose

This component enables Ray in the DataScienceCluster. Ray is a distributed computing framework that makes it easy to scale AI and Python applications, including distributed training and hyperparameter tuning.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-ray
```
