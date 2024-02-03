---
title: "[K8S] Install Nginx Ingress Controller"
# author: wrkholic84
date: 2022-12-26 00:01:00 +0900
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
Nginx 기반 Ingress Controller는 다양한 종류가 있는데 대표적으로 아래 2가지가 있다.

- [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx/)
- [nginxinc/kubernetes-ingress](https://github.com/nginxinc/kubernetes-ingress)

우리는 첫번째 Ingress Controller를 사용한다. (Kubernetes Docs 기준)

### Nginx Ingress Controller 설치
설치 후에 Ingress Controller의 validatingwebhookconfigurations을 삭제. 2020년 3월 업데이트 후 생긴건데, 인증 관련 보안업데이트로 생긴 것으로 보임. 삭제 후 진행.
```bash
ubuntu@cplane:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
# buntu@cplane:~$ kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io ingress-nginx-admission 안해도 됨.
```

### Nginx Ingress Controller 테스트
#### 1.1. 테스트를 위한 Namespace 생성
```bash
ubuntu@cplane:~$ k create ns ingress-test
```

#### 1.2. 테스트용 웹서비스 Deployments로 생성
```bash
ubuntu@cplane:~$ k -n ingress-test create deploy demo-web --image=rancher/hello-world --port 80 --replicas 2
```

#### 1.3. 서비스 생성
target-port 가 rancher/hello-world의 포트
```bash
ubuntu@cplane:~$ kubectl -n ingress-test expose deployment demo-web --name demo-web-svc --port=80 --target-port=80
```

#### 1.4. ingress resource 작성
```bash
ubuntu@cplane:~$ vi ingress-test.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-web-ingress
  namespace: ingress-test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: demo.domain.com   # test 땐 제거
    http:
      paths:
      - path: /test
        pathType: Prefix
        backend:
          service:
            name: demo-web-svc
            port:
              number: 80
```

#### 1.5. ingress 반영
```bash
ubuntu@cplane:~$ k apply -f ./ingress-test.yaml
```

#### 1.6. 접속 테스트
```bash
ubuntu@cplane:~$ k get nodes -o wide
NAME     STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
cplane   Ready    control-plane   4d1h    v1.29.1   172.x.x.x   <none>        Ubuntu 22.04.1 LTS   5.15.0-1019-aws   docker://20.10.12
dplane   Ready    <none>          3d20h   v1.29.1   172.x.x.x    <none>        Ubuntu 22.04.1 LTS   5.15.0-1019-aws   docker://20.10.12
ubuntu@cplane:~$ k -n ingress-nginx get svc
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.97.203.183   <none>        80:31280/TCP,443:31227/TCP   138m
ingress-nginx-controller-admission   ClusterIP   10.97.125.41    <none>        443/TCP                      138m
ubuntu@cplane:~$ curl http://<dplane IP>:31280/test
You've hit demo-web-69d8dc7db6-lppch  # 잘됨
```
### Trouble Shooting
#### 2.1. validatingwebhookconfiguration
```
Error from server (InternalError): error when creating "kubia-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": context deadline exceeded
```
위 에러는 ingress validation 하면서 나는 에러인거 같은데 아래 방법으로 해결한다.
```bash
ubuntu@cplane:~$ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```