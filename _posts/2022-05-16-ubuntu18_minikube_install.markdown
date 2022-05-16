---
layout: custom
title: ubuntu 18.04에 minikube 설치하기
date: 2022-05-16 16:30:00 +0900
last_modified_at: 2022-05-16 16:30:00 +0900
category: minikube
tags: ["ubuntu", "minikube"]
published: true

---
> ubuntu 18.04에 minikube 설치

## 1. minikube 설치
- 공식 문서 "minikube start" 참고
- [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

- minikube 최신 버전 설치
```bash
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
![Untitled](/assets/img/minikube_install_1.png)
    
- 버전 확인
```bash
$ minikube version
```

- 버전 확인 결과
```bash
minikube version: v1.25.2
commit: 362d5fdc...
```

## 2. kubectl 설치
- 공식 문서 "리눅스에 kubectl 설치 및 설정" 참고
- [https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/)

- 최신 버전의 kubectl 바이너리 다운로드
```bash
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

- 바이너리 검증
```bash
$ curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
$ echo "$(<kubectl.sha256)  kubectl" | sha256sum --check
```

- 검증 성공시 `kubectl: OK` 출력
```bash
kubectl: OK
```

- kubectl 설치
```bash
$ sudo install -o root -g root -m 0755 kubectl-convert /usr/local/bin/kubectl-convert
```

## 3. minikube start
- docker driver 기반으로 minikube 시작
```bash
$ minikube start --driver=docker
```
![Untitled](/assets/img/minikube_install_2.png)

- minikube 상태 확인
```bash
minikube status
```

- 상태 확인 결과
```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

- default pod 들이 정상적으로 생성되었는지 확인
```bash
kubectl get pod -n kube-system
```

- 상태 확인 결과
```bash
NAME                               READY   STATUS    RESTARTS        AGE
coredns-64897985d-snc4b            1/1     Running   0               3m10s
etcd-minikube                      1/1     Running   0               3m31s
kube-apiserver-minikube            1/1     Running   0               3m31s
kube-controller-manager-minikube   1/1     Running   0               3m27s
kube-proxy-zb8mv                   1/1     Running   0               3m11s
kube-scheduler-minikube            1/1     Running   0               3m25s
storage-provisioner                1/1     Running   1 (2m33s ago)   3m17s
```