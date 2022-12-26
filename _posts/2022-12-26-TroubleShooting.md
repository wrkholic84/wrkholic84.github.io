---
title: "[K8S] TroubleShooting"
# author: wrkholic84
date: 2022-12-26 00:07:00 +0900
categories: [Development, Kubernetes]
tags: [kubernetes, setup, troubleshooting]
math: true
mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
## 1. CoreDNS 문제

아무리 해도 DNS Query가 안되는 상황이 발생. kube-dns 재시작하니까 됨.

```bash
iMacPro:~$ kubectl delete pod -n kube-system -l k8s-app=kube-dns
```

## Verifying that DNS is working correctly within your Kubernetes platform

Verify that DNS is working correctly within the Kubernetes cluster used by Component Pack for IBM Connections™.

## **About this task**

Before you install Component Pack, you must verify that DNS is working correctly in the Kubernetes cluster, to ensure that pods can communicate with each other over the network.

**Important:** Do not proceed with installing Component Pack until DNS has been verified as working correctly.

## **Procedure**

1. On the Kubernetes master (if you used an HA deployment, select the primary master), set up a test environment by running the following command:
    
    ```
    kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
    ```
    
    This command returns the following result when successful: pod/busybox created.
    
2. Verify that the test pod is running by executing the following command:
    
    `kubectl get pods busybox`
    
    If the pod is running, the command returns the following response: NAME READY STATUS RESTARTS AGE busybox 1/1 Running 0 some-amount-of-time
    
3. Verify that DNS is working correctly by running the following command:
    
    ```
    kubectl exec -ti busybox -- nslookup kubernetes.default
    ```
    
    If DNS is working correctly, the command returns a response like the following example: Server: 10.96.0.10 Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local Name: kubernetes.default Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
    
    Errors such as the following message indicate a problem with DNS: Server: 10.0.0.10 Address 1: 10.0.0.10 nslookup: can't resolve 'kubernetes.default'
    
4. If you encounter a DNS problem, you must correct it.
    
    If the nslookup command fails, execute the following command on all Kubernetes nodes in your cluster (including all masters and worker nodes):
    
    ```
    iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
    ```
    
    Then repeat step 3 to verify that the DNS is now working correctly. If DNS problems persist, recreate the core-dns pods on the Kubernetes master by running the following command (in an HA deployment, execute this command on the primary Kubernetes master):
    
    ```
    kubectl delete pod -n kube-system -l k8s-app=kube-dns
    ```
    
    Remember that you must repeat step 3 to verify that the DNS is now working correctly.
    
    For additional troubleshooting tips, see the [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/) section of the Kubernetes.
    
5. When the DNS is working correctly, delete the test environment by running the following command:
    
    ```
    kubectl delete -f https://k8s.io/examples/admin/dns/busybox.yaml
    ```
    
    This command returns the following response to confirm that the test environment was deleted: pod "busybox" deleted