# components-trustyai

## Purpose

This component enables TrustyAI in the DataScienceCluster. TrustyAI provides explainability (XAI) capabilities for machine learning models, helping you understand and interpret model predictions.

This configuration includes:
- TrustyAI service enabled
- LM Eval settings with code execution and online evaluation denied by default for security

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/components-trustyai
```
