---
layout: single
title: Private Cloud MLOps 실습 자료
date: 2022-05-06 17:31:00 +0900
last_modified_at: 2022-05-06 17:31:00 +0900
category: mlops
tags: ["Private Cloud"]
published: false

---

## Nexus 기반 Private Docker Registry 생성

- Repository 폴더 생성
    
    ```bash
    sudo mkdir -p /nexus-data
    ```
    
- Nexus 컨테이너 생성
    
    ```bash
    docker run --name nexus -d -p 5000:5000 -p 8081:8081 -v /nexus-data:/nexus-data -u root sonatype/nexus3
    ```
    
- Nexus 웹 접속
    - http://<본인 IP>:8081
    - admin 패스워드 확인
        
        ```bash
        cat /nexus-data/admin.password
        ```
        
- [Repository]-[Blob Stores]-[Create Blob Store]
    - docker-hosted
        - Type : File
        - Name : docker-hosted
    - docker-hub
        - Type : File
        - Name : docker-hub
- [Repository]-[Create repository]
    - docker(hosted) 선택
    - HTTP : 5000
    - Blob store : docker-hosted
    - Enable Docker V1 API 체크
- [Repository]-[Create repository]
    - docker(proxy) 선택
    - Enable Docker V1 API 체크
    - Remote storage : https://registry-1.docker.io
    - Docker Index : Use Docker Hub 선택
    - Blob store : docker-hub
- Realms 설정
    - Docker Bearer Token Realm 을 Active 로 변경
- docker 명령어 설정
    - `sudo vim /etc/docker/daemon.json`
        
        ```json
        {
                "insecure-registries" : ["<본인 IP>:5000"]
        }
        ```
        
- docker 권한 변경
    
    ```bash
    sudo chmod 666 /var/run/docker.sock
    ```
    
- docker daemon 재시작
    - `sudo systemctl restart docker`
- Nexus 재시작
    - `docker restart nexus`
- docker 저장소 로그인
    - `docker login <본인 IP>:5000`
- docker 저장소에 push 테스트
    - `docker pull busybox`
    - docker images 로 busybox 의 Image ID 확인
    - `docker tag <docker Image ID> <본인 IP>:5000/busybox:v1`
    - `docker push <본인 IP>:5000/busybox:v1`
- Nexus 에서 확인
    - [Browse]-[docker-hosted 확인]
- docker 저장소에서 pull 받기
    - `docker pull <본인 IP>:5000/busybox:v1`