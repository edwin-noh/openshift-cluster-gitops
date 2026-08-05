# components-workbenches

## Purpose

This component enables Workbenches in the DataScienceCluster. Workbenches provide interactive development environments (based on Jupyter notebooks) for data scientists to develop and test their models.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-workbenches
```
