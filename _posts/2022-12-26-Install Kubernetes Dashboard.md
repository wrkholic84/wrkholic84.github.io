---
title: "[K8S] Install Kubernetes Dashboard"
# author: wrkholic84
date: 2022-12-26 00:02:00 +0900
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
## 1. Deploy Kubernetes Dashboard UI

```bash
ubuntu@master:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```

## 2. Deployment 변경

dashboard는 http 기반으로 동작하며, 로그인 없이 접속 가능하도록 설정.

```yaml
spec:
  containers:
    - args:
      - --enable-skip-login        # 로그인 없이 실행
      - --enable-insecure-login    # http
      - --namespace=kubernetes-dashboard
      image: kubernetesui/dashboard:v2.5.0
      imagePullPolicy: Always
      livenessProbe:
        failureThreshold: 3
        httpGet:
          path: /
          port: 9090
          scheme: HTTP
        initialDelaySeconds: 30
        periodSeconds: 10
        successThreshold: 1
        timeoutSeconds: 3
      name: kubernetes-dashboard
      ports:
      - containerPort: 9090
        protocol: TCP
```

## 3. Service 변경

```yaml
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    k8s-app: kubernetes-dashboard
```

## 4. Ingress 등록

```yaml
ubuntu@master:~$
k -n kubernetes-dashboard get ingress kubernetes-dashboard-ingress -o yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: ### 필요한 도메인
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 9090
```

## 5. RBAC 설정

Dashboard의 ServiceAccount는 클러스터 리소스 접근권한이 없기때문에, Cluster Role의 ‘cluster-admin’ 권한을 바인딩

```bash
# Cluster Role 확인
ubuntu@master:~$ k get clusterrole | grep admin
admin                                                                  2022-11-07T02:03:19Z
**cluster-admin**                                                          2022-11-07T02:03:19Z
system:aggregate-to-admin                                              2022-11-07T02:03:19Z
system:kubelet-api-admin                                               2022-11-07T02:03:19Z
# Dashboard's ServiceAccount 확인
ubuntu@master:~$ k -n kubernetes-dashboard describe deployments.apps kubernetes-dashboard
Name:                   kubernetes-dashboard
...
Pod Template:
  Labels:           k8s-app=kubernetes-dashboard
  Service Account:  **kubernetes-dashboard**
...
# ServiceAccount 확인
ubuntu@master:~$ k -n kubernetes-dashboard get sa  # ServiceAccount 확인
NAME                   SECRETS   AGE
default                0         18m
**kubernetes-dashboard**   0         18m
# ClusterRolebinding 생성 (이 설정은 위험한 설정이다)
ubuntu@master:~$ k create clusterrolebinding kubernetes-dashboard-cluster-admin --serviceaccount kubernetes-dashboard:kubernetes-dashboard --clusterrole cluster-admin
```

## 6. RBAC 설정 2

kubernetes-dashboard에 사용자(ServiceAccount)를 추가하여 이 사용자의 토큰으로 dashboard에 로그인

6.1. admin-user 추가

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

6.2. clusterrolebinding 추가

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

6.3. token 생성

여기서 생성된 토큰을 이용하여 dashboard 에 로그인하면 cluster-admin 권한으로 접근할 수 있다.

```bash
ubuntu@master:~$ kubectl -n kubernetes-dashboard create token admin-user
eyJhbGciOiJSUzI1NiIsImtpZCI6IklMM192eWF...
```