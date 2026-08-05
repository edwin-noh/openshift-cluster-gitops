# components-trainingoperator

## Purpose

This component enables the Training Operator in the DataScienceCluster. The Training Operator (based on Kubeflow Training Operator) provides Kubernetes custom resources for running distributed training jobs using popular ML frameworks like PyTorch, TensorFlow, and others.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-trainingoperator
```
