---
title: "[K8S] Install Kubernetes with Kubeadm"
# author: wrkholic84
date: 2022-12-26 00:00:00 +0900
categories: [Development, Kubernetes]
tags: [kubernetes, setup]
math: true
mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
## Production 환경 구성 : Kubernetes v1.29
학습을 목적으로 EC2(Bare Metal) 환경에 구축
![00](/assets/images/posts/20221226InstallKubernetesWithKubeadm/00.png)
1. Route53 도메인 구입  
2. ALB - Application 연결
3. Public Subnet, Private Subnet 구성  
    - Public Subnet : Control Plane, NAT GW 설치
        > Control Plane이 Public에 있는 이유는 원격 관리자 PC 연결 목적  
        ACM에서 생성한 인증서는 Export가 안되서 Kubernetes API Server의 인증서로 쓸 수 없음
    - Private Subnet : Data Plane 설치
4. EC2 인스턴스 Control Plane, Data Plane 구축
5. Control Plane에 EIP 할당
    - 원격 관리자 PC에서 연결시 필요

***
### Control / Data Plane 초기화
#### 1.1. 원활한 통신을 위한 hostname 설정 (Control / Data Plane 별도 설정 필요)
```bash
ubuntu@cplane:~$ sudo hostnamectl set-hostname cplane
```

#### 1.2. Host 파일 설정 (Control / Data Plane 별도 설정 필요)
```bash
ubuntu@cplane:~$ sudo vi /etc/hosts
127.0.0.1 localhost

172.x.x.x cplane
172.x.x.x dplane
...
```

#### 1.3. 서버 성능 확인 (Optional)
2Core CPU, 2GB 이상 RAM, swap 메모리 비활성화. AWS는 기본적으로 swap 메모리가 비활성화 되어 있음.
```bash
ubuntu@cplane:~$ sudo swapoff -a      // temp
ubuntu@cplane:~$ sudo vi /etc/fstab   // perm
swap 비활성화
# /swap.img     none    swap    sw      0       0

ubuntu@cplane:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.8Gi       170Mi       1.3Gi       0.0Ki       318Mi       1.5Gi
Swap:             0B          0B          0B
```

#### 1.4. iptables가 bridge된 트래픽을 보도록 설정 (Optional)
```bash
root@cplane:~# cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
root@cplane:~# modprobe overlay
root@cplane:~# modprobe br_netfilter
root@cplane:~# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
root@cplane:~# sysctl --system
```

### Container Runtime 설치
#### 2.1. Docker 엔진 설치 (cplane, dplane)
```bash
ubuntu@cplane:~$ sudo apt update
ubuntu@cplane:~$ sudo apt-get install ca-certificates curl
ubuntu@cplane:~$ sudo install -m 0755 -d /etc/apt/keyrings
ubuntu@cplane:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
ubuntu@cplane:~$ sudo chmod a+r /etc/apt/keyrings/docker.asc
ubuntu@cplane:~$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
ubuntu@cplane:~$ sudo apt-get update
ubuntu@cplane:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 2.2. cri-dockerd 설치 (cplane, dplane)
```bash
ubuntu@cplane:~$ sudo apt-get update
ubuntu@cplane:~$ wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.9/cri-dockerd_0.3.9.3-0.ubuntu-jammy_amd64.deb
ubuntu@cplane:~$ sudo dpkg -i ./cri-dockerd_0.3.9.3-0.ubuntu-jammy_amd64.deb
ubuntu@cplane:~$ systemctl status cri-docker
```

### kubeadm, kubelet, kubectl 설치
#### 3.1. apt 패키지 색인 업데이트, kubernetes 패키지 설치 (cplane, dplane)
```bash
ubuntu@cplane:~$ sudo apt-get update
ubuntu@cplane:~$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

#### 3.2. public singing key (cplane, dplane)
```bash
ubuntu@cplane:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

#### 3.3. kubernetes repository 추가 (cplane, dplane)
```bash
ubuntu@cplane:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### 3.4. apt package index update, kubelet, kubeadm, kubectl 설치 (cplane, dplane)
```bash
ubuntu@cplane:~$ sudo apt-get update
ubuntu@cplane:~$ sudo apt-get install -y kubelet kubeadm kubectl
ubuntu@cplane:~$ sudo apt-mark hold kubelet kubeadm kubectl
```

### 클러스터 생성
#### 4.1. Kubeadm을 이용한 Cluster 생성 (cplane)
```bash
ubuntu@cplane:~$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
# kubectl을 사용할 수 있도록 아래 명령 실행 (위 명령의 실행결과로 나옴)
ubuntu@cplane:~$ mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
ubuntu@cplane:~$ kubectl get nodes
NAME     STATUS     ROLES           AGE   VERSION
cplane   **NotReady**   control-plane   15m   v1.29.1
```
kubeadm init 옵션 (추가)  
--control-plane-endpoint=<domain.com,IP 등> : 여러 개의 control-plane 노드로 HA를 구성하는 경우에 사용합니다. 값으로 control-plane 노드들 앞단에 위치한 로드 밸런서의 IP 주소 혹은 도메인 명을 입력  
--apiserver-cert-extra-sans=<domain.com,IP 등> : control-plane 노드가 외부에 위치한 경우(ex, AWS EC2), 로컬 머신에서 kubernetes 클러스터의 API 서버에 접근하기 위해 필요합니다. 해당 옵션은 SSL 인증서의 SAN에 IP, 도메인 명을 추가로 등록

### CNI - Calico 설치
#### 5.1. Calico 설치 (cplane) (참조 : [https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart))
```bash
ubuntu@cplane:~$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
ubuntu@cplane:~$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
ubuntu@cplane:~$ watch kubectl get pods -n calico-system
Every 2.0s: kubectl get pods -n calico-system

NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7f7c5df766-njs4s   1/1     Running   0          65s
calico-node-dkzkm                          1/1     Running   0          65s
calico-typha-69594b6fc6-7k96n              1/1     Running   0          66s
csi-node-driver-djv2n                      2/2     Running   0          65s
```
```bash
ubuntu@cplane:~$ kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
cplane   **Ready**    control-plane   17m   v1.29.1
```

#### 5.2. calicoctl 설치 (cplane)
```bash
ubuntu@cplane:~$ cd /usr/local/bin
ubuntu@cplane:/usr/local/bin$ sudo curl -L https://github.com/projectcalico/calico/releases/download/v3.27.0/calicoctl-linux-amd64 -o calicoctl
ubuntu@cplane:/usr/local/bin$ sudo chmod +x ./calicoctl
```
calico는 기본적으로 Host Route 기능이 없기 때문에 아래와 같은 작업을 해줘야 한다. (참고 : [https://coffeewhale.com/calico-mode](https://coffeewhale.com/calico-mode)) 그런데 calico 3.42 버전에서는 아래와 같이 vxlan이 CrossSubnet으로 활성화되어 있다. 우리는 dplane 노드를 1개 사용하기 때문에 그냥 두도록 한다. 테스트해보니 1개 노드에 있는 여러 파드가 서로 통신하는데는 문제가 없어 보인다. 그래도 라우팅 옵션을 바꾸고 싶다면 6.3 진행.
```bash
ubuntu@cplane:~$ calicoctl get ippool -o wide
NAME                  CIDR             NAT    IPIPMODE   VXLANMODE     DISABLED   DISABLEBGPEXPORT   SELECTOR
default-ipv4-ippool   192.168.0.0/16   true   Never      CrossSubnet   false      false              all()
```

#### 5.4. (Optional) Calico의 라우팅을 IP-in-IP 모드로 변경. 파드간 터널링. 외부(host) 네트워크가 내부(파드) 네트워크를 감싼 형태
```bash
root@cplane:~# cat ./mycni.yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 26
  cidr: 192.168.0.0/16
  ipipMode: Always
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Never
root@cplane:~# calicoctl apply -f ./mycni.yaml
```

### Data Plane 연결
#### 6.1. Control Plane에 Data Plane 연결 (dplane)
```bash
# 4.1. 에서 실행한 명령의 결과로 token 값 출력. cri-socket 옵션 추가 할것.
ubuntu@cplane:~$ sudo kubeadm join 172.x.x.x:6443 --token <token> \
        --discovery-token-ca-cert-hash <sha256:token> \
        --cri-socket unix:///var/run/cri-dockerd.sock
```

### Alias 설정
#### 7.1. kubectl 대신 k 로 명령 실행 (cplane)
```bash
# 꼭해야된다.
ubuntu@cplane:~$ source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
ubuntu@cplane:~$ echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
ubuntu@cplane:~$ echo 'alias k=kubectl' >>~/.bashrc
ubuntu@cplane:~$ echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
```

### 필수 포트 설정
#### 8.1. Control Plane

|프로토콜|방향|포트 범위|용도|사용 주체|
|:---:|:---:|:---:|---|---|
|TCP|인바운드|6443|Kubernetes API Server|All|
|TCP|인바운드|2379-2380|ectd Server Client API|kube-apiserver, etcd|
|TCP|인바운드|10250|Kubelet API|Self, Control Plane|
|TCP|인바운드|10259|kube-scheduler|Self|
|TCP|인바운드|10257|kube-controller-manager|Self|
|TCP|인바운드|179|Calico BGP|Data Plane|
|TCP|인바운드|5473|Calico Networking with Typha|Data Plane|

#### 8.2. Data Plane

|프로토콜|방향|포트 범위|용도|사용 주체|
|:------:|:---:|:---:|---|------|
|TCP|인바운드|10250|Kubelet API|Self, Control Plane|
|TCP|인바운드|30000-32767|NodePort Service|All|