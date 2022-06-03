---
layout: custom
title: nvidia-docker 컨테이너 실행 및 python 설치
date: 2022-06-02 15:38:00 +0900
last_modified_at: 2022-06-02 15:38:00 +0900
category: docker
tags: ["ubuntu", "docker"]
published: false

---
> nvidia-docker 컨테이너 실행 및 python 설치

## 1. minikube 설치
- 지원되는 태그 목록
- [https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md](https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md)

- run
```bash
$ docker run --gpus all --restart=unless-stopped --name {container name} -v {host path}:{container path} -d -p {port}:{port} -p {port}:{port} -it nvidia/cuda:11.4.0-cudnn8-devel-ubuntu18.04
```

- 실행결과
```bash
Unable to find image 'nvidia/cuda:11.4.0-cudnn8-devel-ubuntu18.04' locally
11.4.0-cudnn8-devel-ubuntu18.04: Pulling from nvidia/cuda
40dd5be53814: Pull complete
727501ebf16b: Pull complete
cac242494db3: Pull complete
648bcaa34440: Pull complete
942e9ab9aca4: Pull complete
cd95af8dbbe7: Downloading [=====================>                             ]  539.9MB/1.266GB
7834e91d8511: Download complete
f0cd23b28866: Downloading [==============>                                    ]  500.1MB/1.67GB
4c98ca284304: Download complete
7f3ab65998d9: Downloading [===========>                                       ]  426.5MB/1.898GB

Digest: sha256:e87f0bd9f073bab3bdea0796863dc6a5d335c95a06b5471d32fd44a1f10c271d
Status: Downloaded newer image for nvidia/cuda:11.4.0-cudnn8-devel-ubuntu18.04
bcbc278af02a271b41cd3f73c18574bed9462f08803688bb100920f7c4c08963
```

- 확인
```bash
$ docker ps
```

- 실행
```bash
$ docker exec -it {container name} /bin/bash
```

- 유저 생성(container root)
```bash
$ adduser {user name}
```


- 유저 변경 (container root)
```bash
$ su {user name}
```

- 실행 (host / user 선택)
```bash
$ docker exec -it -u {user name} {container name} /bin/bash
```

```bash
apt-get update && apt-get install vim && apt-get install sudo
visudo -f /etc/sudoers

# User privilege specification
root    ALL=(ALL:ALL) ALL
계정    ALL=(ALL:ALL) ALL

sudo apt install python3 && sudo apt install python3-pip

vi ~/.bashrc
source ~/.bashrc
alias python=python3
alias pip=pip3
sudo apt-get install software-properties-common && sudo add-apt-repository universe && sudo apt-get update

pip install virtualenv
vi ~/.bashrc
PATH="$PATH:$HOME/.local/bin"
source ~/.bashrc

mkdir venv
cd venv
python -m virtualenv tf15
source tf15/bin/activate

pip install tensorflow-gpu==1.15
pip install -U scikit-learn

virtualenv py37_test --python=python3.7
source py37_test/bin/activate
python run_finetuning.py --data-dir /home/kyg/electra_data_dir --model-name electra_small --hparams '{"model_size": "small", "task_names": ["cola"]}'
```


nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04
nvidia/cuda:10.2-cudnn8-devel-ubuntu18.04

10.2-cudnn8-devel-ubuntu18.04: Pulling from nvidia/cuda
40dd5be53814: Already exists
98561051191a: Pull complete
3a18b0f57455: Pull complete
04aea7caabef: Pull complete
d2b1c227b17d: Pull complete
7202c9a06b6b: Downloading [===>                                               ]   48.9MB/759.7MB
bf8621475e5d: Download complete
d730ce521c7a: Downloading [==>                                                ]  52.71MB/881.7MB
a30ddc4b92c5: Download complete
19bee3bd1106: Downloading [=======>                                           ]  112.5MB/745.7MB

Digest: sha256:95aca103f56045f51e1af56c1372f36fb1f9300eb3006faf456de5a394cd2c62
Status: Downloaded newer image for nvidia/cuda:10.2-cudnn8-devel-ubuntu18.04
docker.io/nvidia/cuda:10.2-cudnn8-devel-ubuntu18.04

docker run --gpus all --restart=unless-stopped --name cuda10 -v /home/kyg/mounted/cuda10:/home/mounted -d -p 9013:9013 -p 9014:9014 -p 9015:9015 -it nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04

docker exec -it cuda10 /bin/bash

apt-get update && apt-get install vim && apt-get install sudo
visudo -f /etc/sudoers

# User privilege specification
root    ALL=(ALL:ALL) ALL
kyg     ALL=(ALL:ALL) ALL

adduser kyg
su kyg

sudo apt install python3 && sudo apt install python3-pip

vi ~/.bashrc

alias python=python3
alias pip=pip3

source ~/.bashrc

sudo apt-get install software-properties-common && sudo add-apt-repository universe && sudo apt-get update

pip install virtualenv
vi ~/.bashrc
PATH="$PATH:$HOME/.local/bin"
source ~/.bashrc

mkdir venv
cd venv
python -m virtualenv tf15
source tf15/bin/activate

pip install tensorflow-gpu==1.15 && pip install -U scikit-learn

cd
sudo cp -r ../mounted/electra ./ && sudo cp -r ../mounted/electra_data_dir ./
sudo chown -R kyg:kyg electra && sudo chown -R kyg:kyg electra_data_dir

cd electra
python run_finetuning.py --data-dir /home/kyg/electra_data_dir --model-name electra_small --hparams '{"model_size": "small", "task_names": ["cola"]}'