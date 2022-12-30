---
title: "[K8S] Deployment, Service, Ingress"
# author: wrkholic84
date: 2022-12-26 00:04:00 +0900
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
## Namespace 생성

필요한 Namespace 를 생성

```bash
iMacPro:~$ k create ns domain-dev
iMacPro:~$ k create ns domain-prod
```

## Deployment 생성

Container Repository 에서 이미지를 가져와 Demployment 생성

```bash
iMacPro:~$ k -n domain-dev create deploy domain-web --image=??? --port 8080 --replicas 2
iMacPro:~$ k -n domain-dev create deploy domain-api --image=??? --port 8080 --replicas 2
```

## Service 생성

```bash
iMacPro:~$ k -n domain-dev expose deployment domain-api --name domain-api-svc --port=80 --target-port=8080
iMacPro:~$ k -n domain-dev expose deployment domain-web --name domain-web-svc --port=80 --target-port=8080
```

## Ingress 생성

```bash
iMacPro:~$ vi domain-dev-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: domain-web-ingress
  namespace: domain-dev
spec:
  ingressClassName: nginx
  rules:
  - host: dev.domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: domain-web-svc
            port:
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: domain-api-ingress
  namespace: domain-dev
spec:
  ingressClassName: nginx
  rules:
  - host: dev.domain.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: domain-api-svc
            port:
              number: 80
```