---
layout: custom
title: nvidia-driver, cuda 완전 삭제 및 재설치
date: 2022-08-10 19:59:00 +0900
last_modified_at: 2022-08-10 19:59:00 +0900
category: ubuntu
tags: ["nvidia-driver", "cuda"]
published: false

---
> nvidia-driver, cuda 완전 삭제 및 재설치

## 1. Nvidia driver 완전 삭제

```bash
$ sudo apt-get purge nvidia*
$ sudo apt-get autoremove
$ sudo apt-get autoclean
```

- 에러 발생시
```
Try 'apt --fix-broken install' with no packages (or specify a solution).
```
- 아래 입력 후 삭제 다시 진행
```
$ sudo apt --fix-broken install
```

## 2. cuda 완전 삭제

```bash
$ sudo rm -rf /usr/local/cuda*
$ sudo apt-get --purge remove 'cuda*'
$ sudo apt-get autoremove --purge 'cuda*'
```

## 3. 삭제 확인
- nvidia 확인
```bash
$ sudo dpkg -l | grep nvidia
```
- 삭제가 완료되지 않았으면(확인 결과가 있으면)
```bash
$ sudo apt-get remove --purge {패키지 명}
```

- cuda 확인
```bash
$ sudo dpkg -l | grep cuda
```
- 삭제가 완료되지 않았으면(확인 결과가 있으면)
```bash
$ sudo apt-get remove --purge {패키지 명}
```

## 4. nvidia driver 및 cuda toolkit 설치 (local runfile)
- https://developer.nvidia.com/cuda-toolkit-archive 에서 버전 선택
- 예) linux, x86_64, Ubuntu, 18.04, runfile(local)
```bash
$ wget https://developer.download.nvidia.com/compute/cuda/11.4.0/local_installers/cuda_11.4.0_470.42.01_linux.run
$ sudo sh cuda_11.4.0_470.42.01_linux.run
```

- 설치 중 에러 발생시 로그 확인 후
```bash
$ /var/log/cuda-installer.log
$ /var/log/nvidia-installer.log
```

- 성공시 메세지
```
===========
= Summary =
===========

Driver:   Installed
Toolkit:  Installed in /usr/local/cuda-11.4/
Samples:  Not Selected

Please make sure that
 -   PATH includes /usr/local/cuda-11.4/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-11.4/lib64, or, add /usr/local/cuda-11.4/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.4/bin
To uninstall the NVIDIA Driver, run nvidia-uninstall
Logfile is /var/log/cuda-installer.log
```

- CUDA bin 경로 `/etc/profile`에 추가 및 CUDA lib 경로 `/etc/ld.so.conf`에 추가
```
# /etc/profile
export PATH=$PATH:/usr/local/cuda/bin
export CUDADIR=/usr/local/cuda

# /etc/ld.so.conf
include /usr/local/cuda/lib64
```

## 5. cuDNN 설치
- https://developer.nvidia.com/rdp/cudnn-archive
- 로그인 -> 원하는 버전 선택 -> 아래 선택하여 다운로드
```
cuDNN Library for Linux [x86_64]
```

- 압축 해제 및 파일 복사
```
$ tar -xvf cudnn-11.3-linux-x64-v8.2.1.32.tgz
$ sudo cp ./cuda/include/cudnn*.h /usr/local/cuda/include 
$ sudo cp -P ./cuda/lib64/libcudnn* /usr/local/cuda/lib64 
$ sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

## 6. ubuntu 커널 업데이트 후 nvidia-smi를 할 수 없는 경우
```
$ sudo sh NVIDIA-Linux-x86_64-470.42.01.run --dkms
```