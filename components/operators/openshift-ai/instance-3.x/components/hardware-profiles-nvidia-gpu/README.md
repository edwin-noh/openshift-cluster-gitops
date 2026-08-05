# hardware-profiles-nvidia-gpu

## Purpose

This component configures hardware profiles for NVIDIA GPUs. Hardware profiles define resource constraints and scheduling requirements for workloads that require GPU acceleration.

This component includes two hardware profiles:
- **nvidia-gpu-serving**: For model serving workloads (visible in model-serving and pipelines)
- **nvidia-gpu-workbench**: For workbench environments (visible in workbenches and pipelines)

Both profiles include:
- CPU: 1-32 cores (default: 4)
- Memory: 2Gi-120Gi (default: 8Gi)
- GPU: 1-8 NVIDIA GPUs (default: 1)
- Node tolerations for `nvidia-gpu-only` and `nvidia.com/gpu` taints

For more information on hardware profiles, see the [Working with hardware profiles](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2-latest/html/working_with_accelerators/working-with-hardware-profiles_accelerators) documentation.

## Usage

This component can be added to a base by adding the `components` section to your overlay `kustomization.yaml` file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/hardware-profiles-nvidia-gpu
```

You can customize the hardware profiles by modifying:
- [hardware-profile-serving.yaml](./hardware-profile-serving.yaml)
- [hardware-profile-workbench.yaml](./hardware-profile-workbench.yaml)
