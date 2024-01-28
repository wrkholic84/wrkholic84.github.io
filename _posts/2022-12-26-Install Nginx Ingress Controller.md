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

## Nginx Ingress Controller 설치

설치 후에 Ingress Controller의 validatingwebhookconfigurations을 삭제. 2020년 3월 업데이트 후 생긴건데, 인증 관련 보안업데이트로 생긴 것으로 보임. 삭제 후 진행.

```bash
ubuntu@master:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/baremetal/deploy.yaml
ubuntu@master:~$ kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io ingress-nginx-admission
```

## Nginx Ingress Controller 테스트

2.1. 테스트를 위한 Namespace 생성

```bash
ubuntu@master:~$ k create ns ingress-test
```

2.2. 테스트용 웹서비스 Deployments로 생성

```bash
ubuntu@master:~$ k -n ingress-test create deploy demo-web --image=whatwant/node-web:1.0 --port 8080 --replicas 2
```

2.3. 서비스 생성

```bash
ubuntu@master:~$ kubectl -n ingress-test expose deployment demo-web --name demo-web-svc --port=80 --target-port=8080
```

2.4. ingress resource 작성

```bash
ubuntu@master:~$ vi ingress-test.yaml
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
  - host: demo.domain.com
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

2.5. ingress 반영

```bash
ubuntu@master:~$ k apply -f ./ingress-test.yaml
```

2.6. 접속 테스트

```bash
ubuntu@master:~$ k get nodes -o wide
NAME     STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
master   Ready    control-plane   4d1h    v1.25.3   172.x.x.x   <none>        Ubuntu 22.04.1 LTS   5.15.0-1019-aws   docker://20.10.12
worker   Ready    <none>          3d20h   v1.25.3   172.x.x.x    <none>        Ubuntu 22.04.1 LTS   5.15.0-1019-aws   docker://20.10.12
ubuntu@master:~$ k -n ingress-nginx get svc
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.97.203.183   <none>        80:31280/TCP,443:31227/TCP   138m
ingress-nginx-controller-admission   ClusterIP   10.97.125.41    <none>        443/TCP                      138m
ubuntu@master:~$ curl http://172.31.20.49:31280/test
You've hit demo-web-69d8dc7db6-lppch  # 잘됨
```

## AWS ALB 설정

3.1. AWS에서 로드밸런서의 대상그룹에 Worker Node 로 사용되는 EC2 등록. 포트는 31280.