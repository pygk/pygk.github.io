---
layout: custom
title: ubuntu 18.04에 nvidia-docker2 설치하기
date: 2022-05-11 17:15:00 +0900
last_modified_at: 2022-05-11 17:15:00 +0900
category: docker
tags: ["ubuntu", "docker", "nvidia-docker2"]
published: true

---
> ubuntu 18.04에 nvidia-docker2 설치

## 1. nvidia-docker2 설치
- 공식 문서 Installation Guide 참고
- [https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

- Docker 설정
```bash
$ curl https://get.docker.com | sh \
  && sudo systemctl --now enable docker
```
    
- `apt` 패키지 업데이트 & 설치
```bash
$ sudo apt-get update
$ sudo apt-get install -y nvidia-docker2
```

- Docker daemon 재시작
```bash
$ sudo systemctl restart docker
```

- 확인
```bash
$ sudo docker run --rm --gpus all nvidia/cuda:11.4.0-base-ubuntu20.04 nvidia-smi
```
![Untitled](/assets/img/nvidiadocker_install_1.png)