---
layout: single
title: ubuntu 18.14 vscode install
date: 2022-03-06 13:28:00 +0900
last_modified_at: 2022-03-06 13:28:00 +0900
category: blog
tags: ["tag1", "tag2"]
---
> ubuntu 18.14 vscode install

## 1. curl install
```
$ sudo apt-get install curl
```

## 2. ms GPG download 마이크로소프트 GPG 키를 다운로드하여 /etc/apt/trusted.gpg.d/ 경로에 복사
```
$ sudo sh -c 'curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg'
```

## 3. Visual Studio Code를 다운로드 받기 위한 저장소를 추가
```
$ sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
```

## 4. 추가한 저장소로부터 패키지 목록을 가져옵니
```
$ sudo apt update
```

## 5. Visual Studio Code를 설치
```
$ sudo apt install code
```

## 6. vscode 실행
```
code
```
or
Show applications -> search 'vscode'

### Reference
Visual Studio Code 설치하는 방법( Windows / Ubuntu ) (https://webnautes.tistory.com/1197)