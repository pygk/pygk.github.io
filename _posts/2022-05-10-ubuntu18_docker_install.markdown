---
layout: custom
title: ubuntu 18.04에 Docker 설치하기
date: 2022-05-10 15:22:00 +0900
last_modified_at: 2022-05-10 15:22:00 +0900
category: docker
tags: ["ubuntu", "docker"]
published: true

---
> ubuntu 18.04에 Docker 설치 & 설치 이후 설정

## 1. Docker 설치
- 공식 문서 Install using the repository 참고
- [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

### 1) repository 설정
- `apt` 패키지 업데이트
```bash
$ sudo apt-get update
```
    
- 패키지 설치
```bash
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

- Docker의 공식 GPG 키 추가
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

- Repository 설정
```bash
$ echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 2) Docker 엔진 설치
- `apt` 패키지 업데이트 & 최신 버전의 Docker 엔진 설치
```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- 설치 확인
```bash
$ sudo docker run hello-world
```
![Untitled](/assets/img/docker_install_3.png)


## 2. 설치 이후 단계
- 공식 문서 Post-installation steps for Linux 참고
- [https://docs.docker.com/engine/install/linux-postinstall/](https://docs.docker.com/engine/install/linux-postinstall/)

### 1) root가 아닌 사용자로 Docker 관리
- Docker daemon은 Unix socket에 바인딩되고, Unix socket은 `root`가 소유함. 다른 유저는 `sudo`를 통해서만 접근 가능.
- `docker` unix gourp을 만들고 사용자를 추가하여 `sudo` 없이 `docker` 명령을 사용
    - Docker damon이 시작되면 `docker` group의 멤버가 액세스할 수 있는 유닉스 소켓이 생성됨

- group 생성
```bash
sudo groupadd docker
```

- `Docker` group에 사용자 추가
```bash
$ sudo usermod -aG docker $USER
```

- group에 대한 변경사항 활성화
```bash
$ newgrp docker
```

- 확인
```bash
$ docker run hello-world
```

### 2) 부팅 시 시작
- 부팅 시 Docker 및 Containerd를 자동으로 시작
```bash
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```

- 비활성화
```bash
$ sudo systemctl disable docker.service
$ sudo systemctl disable containerd.service
```