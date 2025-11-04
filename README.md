# Edwin's OpenShift GitOps

## 컨셉
1. 다양한 용도와 버전의 클러스터 인스턴스를 필요에 따라 bootstrapping 한다. (RHOAI, RHADS, kafka클러스터 등)
2. 초기 스크립트는 Ansible을 통해 수행되며 이 후는 OpenShift GitOps(ArgoCD)를 통해 provisioning
3. 각 버전의 클러스터에 오퍼레이터 설치는 간략하게 설정할 수 있도록 공통화

## Pre-requsite
1. OpenShift 설치는 이전 단계에서 완료되었음
2. bootstraping 스크립트를 통해 openshift-gitops operator와 argocd instance는 설치되었음 (components/operator/openshift-gitops)
3. IaC를 위한 git repository 연결은 완료되었음
(>> 이건 어디에서?)
## 기본 구조
1. bootstrap
2. clusters
3. operators
- 기본적인 operator는 
## 클러스터 정의
0. 공통
- operator
  - base : 기본적인 subscription
  - components : 컴포넌트 설정
  - overlays : 버전 오버레이

- ** openshift-gitops **
1. Operator 설치
/components/operators/openshift-gitops/operator 에서 설치
2. git연결(github에서는 필요없음) 및 controller권한 설정
/components/operators/openshift-gitops/components 에서 설정
** 이건 부트스트랩에서 설정 **
3. nfs-provisioner
> [!WARNING]
> 클러스터 별로 sub-directory를 구별

## 클러스터 별 설정
### 1. rhocp-app

- openshift-pipeline
- openshift-
기본 오픈시프트 
# Red Hat OpenShift Day2 GitOps

## Usecase
공통 부분과 오버레이 부분을 식별하기 위해 몇가지 유스케이스를 살펴본다.
예를 들어, 일반적인 클러스터를 설정하는 과정
1. 오픈시프트 설정
- OpenShift GitOps Operator 설치
- ArgoCD instance 설치
- ArgoCD 설정 (AppProject, RBAC, Repo, Notification, Plugins)
- 필요 Operator 설치 및 설정 (Tekton, Service Mesh 등) **cluster config applicationset**등과 같이 ArgoCD를 통해 설치
- 필요 컴포넌트 설정 (NFS Provisioner 등)  **cluster config applicationset**등과 같이 ArgoCD를 통해 설치
2. RHOAI 설정
분기되는 부분만 확인하면
필요 Operator 설치, 설정이 틀리고...
MinIO등 필요 컴포넌트 설정이 틀림
3. 같은 오픈시프트 구성이지만 버전이 틀린 경우
전체 과정에서 GitOps등과 같이 오퍼레이터 버전 등이 틀림

**결론**
클러스터 타입과 버전별로 오버레이 구성

## Prerequisite
1. Create namespace
``` bash
% oc create ns openshift-gitops-operator
# Optional
% oc label namespace openshift-gitops-operator openshift.io/cluster-monitoring=true
```
2. Create Operator group
``` bash
cat <<EOF > "gitops-operator-group.yaml"
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
EOF

oc apply -f gitops-operator-group.yaml
```
3. Create subscription
``` bash
cat <<EOF > "openshift-gitops-sub.yaml"
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: latest 
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

oc oc apply -f openshift-gitops-sub.yaml
```


## Design Concept

### 1. BootStrap 
1.1. Base - Bootstrap process is installing argocd instnace and argocd components
1.2. Overlays - selecting cluster. i.e. rhoai, application, hub cluster.
### 2. Cluster
2.1. Base - basic setting of cluster...maybe app project of argocd, nfs provisioner...
2.2. Overlays - Cluster overlays have a set of operators and installation manifest to enable cluster for the purpose via applicationset. 
### 3. 
Components have operators and tools for the clusters

Git repo to install and configure RHOAI in GitOps way through ArgoCD.

``` sh

```