---
title: "[K8S] Cluster Setup"
# author: wrkholic84
date: 2022-12-26 00:03:00 +0900
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
## 운영자 PC에 클러스터 설정

1.1. kubectl 설치 (Mac 기준)

```bash
iMacPro:~$ brew install kubectl
```

1.2. Master 노드에 개발용 인증서 생성 (optional)

```bash
ubuntu@master:/etc/kubernetes/pki$ sudo openssl genrsa -out domain.com.ca.key 2048
ubuntu@master:/etc/kubernetes/pki$ sudo openssl req -x509 -new -nodes -key ./domain.com.ca.key -subj "/CN=domain.com" -addext "subjectAltName = DNS:domain.com,DNS:domain.com.default,DNS:domain.com.default.svc,DNS:domain.com.default.svc.cluster.local,DNS:master,IP:10.96.0.1,IP:172.31.28.166" -out domain.com.ca.crt
```

1.3. Master 노드에 클러스터 등록 (optional)

apiserver.crt 인증서를 사용하는 이유는, 클러스터와 통신한다는 것이 결국 apiserver와 통신을 의미하기때문이다. 따라서 apiserver의 인증서를 사용한다.

이 인증서를 위에서 만든 개발용 인증서로 사용하고 싶다면, kube-apiserver.yaml에서 -tls-cert-file과 -tls-private-key-file 옵션을 변경해줘야한다.

```bash
ubuntu@master:~$ k config set-cluster domain.com --server https://172.x.x.x:6443 --embed-certs --certificate-authority=/etc/kubernetes/pki/domain.com.ca.crt
```

1.4. 운영자 PC 호스트파일 변경

Master 노드에서 apiserver.crt(/etc/kubernetes/pki/)를 다운로드 받은 후, 아래 명령으로 정보 확인.

클러스터를 등록하면서 사용한 인증서(crt) 파일은 kubernetes를 호스트로 요청 시에만 사용할 수 있다.

```shell
iMacPro:~$ openssl x509 -in ./apiserver.crt -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        ...
        **Issuer: CN = kubernetes**
        ...
        X509v3 extensions:
            ...
            **X509v3 Subject Alternative Name:
                DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:master, IP Address:10.96.0.1, IP Address:172.x.x.x**
    Signature Algorithm: sha256WithRSAEncryption
    ...
```

별도로 인증서를 바꾸지 않고, 기본적으로 생성된 인증서(apiserver.crt)를 사용할 경우 아래와 같이 운영자 PC의 호스트 파일에 kubernetes를 아래와 같이 등록하여 사용하면 된다.

```shell
/etc/hosts
...
13.206.22.22  kubernetes
...
```

1.5. 운영자 PC에 클러스터 등록

```shell
iMacPro:~$ k config set-cluster domain.com --server https://kubernetes:6443 --embed-certs --certificate-authority=./apiserver.crt
```

## 개발자(운영) 계정 추가

운영자 계정으로, Dev,Stg,Prod Namespace에서 동시에 사용할 예정

2.1. 개발자 계정 인증서 생성

```bash
ubuntu@master:~/domain$ openssl genrsa -out domain-ops.key 2048
ubuntu@master:~/domain$ openssl req -new -key domain-ops.key -out domain-ops.csr
...
Common Name (e.g. server FQDN or YOUR name) []:**domain-ops**
...
```

2.2. CSR 생성

```bash
ubuntu@master:~/domain$ cat ./domain-ops.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: domain-ops
spec:
  request: LS0tLS1CRU...  # cat ./domain-ops.csr | base64 | tr -d "\n" 
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
ubuntu@master:~/domain$ k apply -f ./domain-ops.yaml
```

2.3. CSR 승인

```bash
ubuntu@master:~/domain$ k get csr
NAME        AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
domain-ops   5s    kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              **Pending**
ubuntu@master:~/domain$ k certificate approve domain-ops
ubuntu@master:~/domain$ k get csr
NAME        AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
domain-ops   2m34s   kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              **Approved,Issued**
```

2.4. 인증서 추출

```bash
ubuntu@master:~/domain$ k get csr domain-ops -o jsonpath='{.status.certificate}' | base64 -d > domain-ops.crt
```

2.5. 개발자 계정 kubeconfig에 등록

```bash
ubuntu@master:~/domain$ k config set-credentials domain-ops --client-key ./domain-ops.key --client-certificate ./domain-ops.crt --embed-certs=true
```

2.6. 운영자PC에 개발자 계정 등록. 위에서 만든 사용자 키, 인증서는 서버로부터 다운로드 받아온다.

```bash
iMacPro:~$ k config set-credentials domain-ops --client-key ./domain-ops.key --client-certificate ./domain-ops.crt --embed-certs=true
```

2.7. 운영자PC에 컨텍스트 생성

```bash
iMacPro:~$ k config set-context domain-ops@domain.com --cluster domain.com --user domain-ops
```

## 운영자 PC에 최종. 추가된 클러스터, 사용자, 컨텍스트 정보

```bash
iMacPro:~$ k config view
apiVersion: v1
clusters:
**- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes:6443
  name: domain.com**
...
contexts:
...
**- context:
    cluster: domain.com
    user: domain-ops
  name: domain-ops@domain.com**
current-context: domain-ops@domain.com
kind: Config
preferences: {}
users:
...
**- name: domain-ops
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED**
```

## 개발자(운영) 계정 RBAC 추가

3.1. 개발자(운영)에게 필요한 role 확인

우선, 모든 권한을 부여해주자.

```bash
ubuntu@master:~$ k create clusterrolebinding domain-ops --clusterrole cluster-admin --user domain-ops
```

클러스터, 리소스별 권한은 차차 부여하기로 한다.