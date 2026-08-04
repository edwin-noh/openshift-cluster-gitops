# 오픈시프트 SNO 및 GitOps 자동 부트스트랩 구축 결과

RHEL 8 KVM 서버 환경에서 200GB 디스크와 물리 Bridge(`br0`) 네트워크를 연동하여 OpenShift 4.16 SNO(Single Node OpenShift)를 자동으로 설치하고, 최종적으로 OpenShift GitOps(Argo CD)를 통해 현재 코드베이스의 `bootstrap` 디렉토리를 셀프 프로비저닝(부트스트랩)하도록 연동하는 Ansible 자동화 아키텍처가 완전히 구현되었습니다.

---

## 🏗️ 제공된 자동화 아키텍처 구성

새롭게 추가된 `ansible/` 폴더 하위에 견고한 K8s 인프라스트럭처 자동화 프레임워크가 구성되었습니다.

```
ansible/
├── ansible.cfg              # 플레이북 실행을 위한 최적 환경 튜닝
├── inventory.ini            # RHEL 8 하이퍼바이저 접속용 인벤토리 정의
├── playbook.yml             # 전체 리소스 생성의 컨트롤타워
├── vars/
│   ├── main.yml             # OCP 4.16, br0, SNO IP(192.168.31.30), 200GB 볼륨 등 비민감성 변수
│   └── vault.yml            # Red Hat Pull Secret, SSH key 등 민감 정보를 암호화 관리할 템플릿
└── roles/
    ├── kvm_setup/           # RHEL 8 가상화 패키징, 200G 볼륨 풀, Bridge 가상 네트워크 연동
    ├── openshift_sno/       # 최신 4.16 CLI 다운로드, ISO 부팅 이미지 빌드, VM 기동 및 대기
    └── openshift_gitops/    # GitOps Operator 배포 및 bootstrap Application 자동 연동
```

---

## 🚀 플레이북 실행 및 구축 가이드

실제 KVM 서버에 OpenShift SNO를 배포하기 위해 아래 단계를 진행해 주세요.

### 1단계: RHEL 8 접속 정보 기재
`ansible/inventory.ini` 파일을 열어 KVM을 구동할 RHEL 8 하이퍼바이저 서버의 실제 IP와 접속 계정을 수정합니다.
```ini
[hypervisors]
kvm-host ansible_host=192.168.31.10 ansible_user=root
```

### 2단계: Ansible Vault에 자격 증명 설정
Red Hat Portal에서 다운로드받은 Pull Secret과 SNO에 주입할 SSH 공개키 정보를 `ansible/vars/vault.yml` 파일에 입력한 뒤, 이를 안전하게 암호화합니다.
```bash
# 1. 파일 내의 플레이스홀더에 실제 데이터를 기입합니다.
ssh-keygen -t ed25519 -N '' -f /Users/edwin/Workspace/my-keys/edwin-ocp

vi ansible/vars/vault.yml

# 2. ansible-vault를 활용하여 파일을 강력하게 암호화합니다.
ansible-vault encrypt ansible/vars/vault.yml
```
*(암호화 시 지정할 패스워드를 잘 기억해 두시기 바랍니다.)*

### 3단계: GitOps 변수 커스텀 설정
`ansible/vars/main.yml` 파일을 열고, GitOps 부트스트랩 연동을 진행할 **사용자 본인의 Git Repository URL**을 기입합니다.
```yaml
# Argo CD에 자신을 등록할 Git 리포지토리 URL 및 타겟 브랜치
gitops_repo_url: "https://github.com/your-username/openshift-cluster-gitops.git"
gitops_repo_branch: "main"
gitops_bootstrap_path: "bootstrap/overlays/rhocp-4.20"
```

### 4단계: 플레이북 구동
모든 설정이 완료되면 아래 명령어로 암호화 암호를 제공하며 플레이북을 실행합니다.
```bash
ansible-playbook -i ansible/inventory.ini ansible/playbook.yml --ask-vault-pass
```

---

## 🔍 프로비저닝 워크플로우 세부 동작 설명

1. **인프라스트럭처 기동 (`kvm_setup`)**
   - 하이퍼바이저에 `qemu-kvm`, `libvirt`, `virt-install` 등을 빌드 및 활성화합니다.
   - `/var/lib/libvirt/images/openshift-pool` 경로로 Storage Pool을 구성하고, SNO의 가상 하드디스크 역할을 수행할 **200GB 크기의 QCOW2 이미지 볼륨**을 확보합니다.
   - `br0` 물리 브릿지 어댑터와 VM 네트워크 카드를 결합하기 위한 libvirt 가상 Bridge 네트워크(`host-bridge`)를 자동 매핑합니다.
2. **SNO 배포 (`openshift_sno`)**
   - 4.16 버전의 안정화 배포판 CLI 툴들을 하이퍼바이저 로컬에 다운로드합니다.
   - `agent-config.yaml` nmstate 스펙을 통해 가상머신의 네트워크 카드가 구동 시 고정 IP인 `192.168.31.30` 및 Gateway `192.168.31.1`, DNS `8.8.8.8`로 세팅되도록 정적 맵핑합니다.
   - `openshift-install agent create image`를 통해 SNO 무인 배포를 위한 **부팅 전용 Agent ISO**를 직접 빌드합니다.
   - `virt-install`을 이용하여 **8 vCPU, 32GB RAM, 200GB 스토리지** 사양의 SNO VM을 론칭하고 설치가 최종 끝날 때까지 로그를 스트리밍 추적 대기합니다.
3. **Argo CD 통합 (`openshift_gitops`)**
   - SNO 설치 종료 즉시 클러스터 권한(`kubeconfig`)을 획득하여 `openshift-gitops-operator`를 가동합니다.
   - Argo CD 서비스 Pod가 Ready 상태로 승격되면, 앞서 지정한 사용자의 GitOps 리포지토리 경로(`bootstrap/overlays/rhocp-4.20`)를 Argo CD self-healing Application으로 주입하여, 이 Git 리포지토리의 모든 클러스터 Day-2 설정(Apps, Configs, Operators)이 연쇄 배포되도록 GitOps 부트스트랩을 출범합니다.
