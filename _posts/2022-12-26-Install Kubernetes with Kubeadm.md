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
## 1. Production 환경 구성

1.1. 노드간 원활한 통신을 위한 hostname 설정 (Master; worker 별도 설정 필요)

```bash
root@ip:~# hostnamectl set-hostname master
```

1.2. Host 파일 설정 (Master; worker 별도 설정 필요)

```bash
root@ip:~# vi /etc/hosts
127.0.0.1 localhost

172.x.x.x master
172.x.x.x worker
...
```

1.3. 노드 확인 - 2Core CPU, 2GB 이상 RAM, swap 메모리 비활성화. AWS는 기본적으로 swap 메모리가 비활성화 되어 있음.

```bash
ubuntu@master:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.8Gi       170Mi       1.3Gi       0.0Ki       318Mi       1.5Gi
Swap:             0B          0B          0B
```

1.4. iptables가 bridge된 트래픽을 보도록 설정 (Master, Worker)

```bash
root@master:~# cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
root@master:~# modprobe overlay
root@master:~# modprobe br_netfilter
root@master:~# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
root@master:~# sysctl --system
```

## 2. Container Runtime 설치

2.1. Docker 엔진 설치 (Master, Worker)

```bash
root@master:~# apt update
root@master:~# apt install -y docker.io
root@master:~# systemctl enable --now docker
```

2.2. cri-dockerd 설치 (Master, Worker)

```bash
root@master:~# cat /etc/os-release  # OS 버전 확인
root@master:~# wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.6/cri-dockerd_0.2.6.3-0.ubuntu-jammy_amd64.deb
root@master:~# dpkg -i ./cri-dockerd_0.2.6.3-0.ubuntu-jammy_amd64.deb
root@master:~# systemctl status cri-docker
```

## 3. kubeadm, kubelet, kubectl 설치

3.1. apt 패키지 색인 업데이트, kubernetes 패키지 설치 (Master, Worker)

```bash
root@master:~# sudo apt-get update
root@master:~# sudo apt-get install -y apt-transport-https ca-certificates curl
```

3.2. 구글 클라우드 public singing key (Master, Worker)

```bash
root@master:~# sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

3.3. kubernetes repository 추가 (Master, Worker)

```bash
root@master:~# echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

3.4. apt package index update, kubelet, kubeadm, kubectl 설치 (Master, Worker)

```bash
root@master:~# sudo apt-get update
root@master:~# sudo apt-get install -y kubelet kubeadm kubectl
root@master:~# sudo apt-mark hold kubelet kubeadm kubectl
```

## 4. 클러스터 생성

4.1. Kubeadm을 이용한 Cluster 생성 (Master)

```bash
root@master:~# kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
# kubectl을 사용할 수 있도록 아래 명령 실행 (위 명령의 실행결과로 나옴)
root@master:~# mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@master:~# kubectl get nodes
NAME     STATUS     ROLES           AGE   VERSION
master   **NotReady**   control-plane   15m   v1.25.3
```

## 5. CNI - Calico 설치

5.1. Calico 설치 (참조 : [https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart))

```bash
root@master:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.4/manifests/tigera-operator.yaml
root@master:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.4/manifests/custom-resources.yaml
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
master   **Ready**    control-plane   17m   v1.25.3
```

## 6. Calico 라우팅 설정

6.1. calicoctl 설치

```bash
root@master:~# cd /usr/local/bin
root@master:/usr/local/bin# curl -L https://github.com/projectcalico/calico/releases/download/v3.24.4/calicoctl-linux-amd64 -o calicoctl
root@master:/usr/local/bin# chmod +x ./calicoctl
```

6.2. calico는 기본적으로 Host Route 기능이 없기 때문에 아래와 같은 작업을 해줘야 한다. (참고 : [https://coffeewhale.com/calico-mode](https://coffeewhale.com/calico-mode)) 그런데 calico 3.42 버전에서는 아래와 같이 vxlan이 CrossSubnet으로 활성화되어 있다. 우리는 worker 노드를 1개 사용하기 때문에 그냥 두도록 한다. 테스트해보니 1개 노드에 있는 여러 파드가 서로 통신하는데는 문제가 없어 보인다. 그래도 라우팅 옵션을 바꾸고 싶다면 6.3 진행.

```bash
ubuntu@master:~$ calicoctl get ippool -o wide
NAME                  CIDR             NAT    IPIPMODE   VXLANMODE     DISABLED   DISABLEBGPEXPORT   SELECTOR
default-ipv4-ippool   192.168.0.0/16   true   Never      CrossSubnet   false      false              all()
```

6.3. (Optional) Calico의 라우팅을 IP-in-IP 모드로 변경. 파드간 터널링. 외부(host) 네트워크가 내부(파드) 네트워크를 감싼 형태

```bash
root@master:~# cat ./mycni.yaml
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
root@master:~# calicoctl apply -f ./mycni.yaml
```

## 7. Worker Node 연결

7.1. Master Node에 Worker Node 연결

```bash
# 4.1. 에서 실행한 명령의 결과로 token 값 출력. cri-socket 옵션 추가 할것.
root@master:~# kubeadm join 172.x.x.x:6443 --token <token> \
        --discovery-token-ca-cert-hash <sha256:token> \
        --cri-socket unix:///var/run/cri-dockerd.sock
```