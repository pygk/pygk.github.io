---
layout: single
title: 실습 3. Python 기반 Jenkins CI Pipeline 생성 실습
date: 2022-05-06 16:57:00 +0900
last_modified_at: 2022-05-06 16:57:00 +0900
category: mlops
tags: ["Jenkins for MLOps"]
published: false

---

<aside>
💡 **실습 목표
1. 앞에서 배운 Jenkins CI Pipeline 생성을 Python 어플리케이션에 적용해 본다.
2. Github 에 Push 시 자동으로 배포하는 trigger 를 설정해 본다.
3. 배포된 Docker image 를 Docker Hub 로 올리기**

</aside>

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3bc9519e-153e-4a4f-b5d2-3b4a9d035bd7/Untitled.png) -->
![Untitled](/assets/img/mlops_jenkins3_1.png)

### **앞에서 배운 Jenkins CI Pipeline 생성을 Python 어플리케이션에 적용해 본다.**

- FastAPI 예제 코드를 생성하여 서버에서 Docker Container 를 실행해 본다.
    - app/main.py
        
        ```python
        from fastapi import FastAPI
        
        app = FastAPI()
        
        @app.get("/")
        def read_root():
            return {"Hello": "MLOps"}
        
        @app.get("/items/{item_id}")
        def read_item(item_id: int, q: str = None):
            return {"item_id": item_id, "q": q}
        ```
        
    - app/requirements.txt
        
        ```python
        fastapi
        uvicorn
        ```
        
    - 서버에 docker-compose 설치
    
    ```bash
    sudo apt install docker-compose
    ```
    
    - Dockerfile 작성
    
    ```docker
    FROM python:3.9
    
    WORKDIR /app
    
    COPY ./app/requirements.txt /app/requirements.txt
    
    RUN pip install --no-cache-dir --upgrade -r /app/requirements.txt
    
    COPY ./app /app
    
    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
    ```
    
    - docker-compose.yml 작성
    
    ```docker
    version: "3"
    
    services:
      web:
        build: .
        container_name: fastapi-app
        volumes:
          - .:/code
        ports:
          - "80:80"
    ```
    
    - 서버에서 Docker Container 생성해 보기
    
    ```bash
    docker-compose build web
    docker images
    docker-compose up -d
    docker ps -a
    ```
    
    - [localhost:80](http://localhost:80) 접속하여 확인
- Jenkinsfile 을 작성하여 Jenkins 에서 배포해 본다.
    - docker group 에 jenkins 등록
        - `sudo gpasswd -a jenkins docker`
        - `sudo vi /usr/lib/systemd/system/docker.service` (에러 발생 시에만 시도)
            
            ```python
            ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --containerd=/run/containerd/containerd.sock
            ```
            
        - `sudo systemctl daemon-reload`
        - `sudo systemctl restart docker`
        - `sudo service jenkins restart`
    
    ```yaml
    pipeline {
    	agent any
    	parameters {
    		choice(name: 'VERSION', choices: ['1.1.0','1.2.0','1.3.0'], description: '')
    		booleanParam(name: 'executeTests', defaultValue: true, description: '')
    	}
    	stages {
    		stage("init") {
    			steps {
    				script {
    					gv = load "script.groovy"
    				}
    			}
    		}
    		stage("Checkout") {
    			steps {
    				checkout scm
    			}
    		}
    		stage("Build") {
    			steps {
    				sh 'docker-compose build web'
    			}
    		}
    		stage("test") {
    			when {
    				expression {
    					params.executeTests
    				}
    			}
    			steps {
    				script {
    					gv.testApp()
    				}
    			}
    		}
    		stage("deploy") {
    			steps {
    				sh "docker-compose up -d"
    			}
    		}
    	}
    }
    ```
    

### **Github 에 Push 시 자동으로 배포하는 trigger 를 설정해 본다.**

- Jenkins pipeline 에 Github 을 바로 가도록 설정해 본다.
- Poll SCM 은 매 시간마다 소스가 변경되었는지 확인한다.
    - 예) H/3 * * * * → 3분마다 소스가 변경되었는지 확인
- 우리는 'GitHub hook trigger for GITScm polling' 선택
- Github Webhook 설정을 위한 VirtualBox  네트워크 설정 변경
    - [설정]-[네트워크]-[어댑터에 브리지]로 변경-[가상머신 재시작]
    - `sudo service jenkins restart`
    - ifconfig 명령으로 public ip 확인
    - Jenkins 로 배포하여 접근 확인
- Github Webhook 설정
    - Github Repository - Settings - Webhooks - Add webhook
    - Payload URL : [http://<VirtualBox Public IP>:8080/github-webhook/](http://114.203.232.71:8080/github-webhook/)
    - Content type : application/json
    - Acitve 활성화
    - 코드 변경 후 push 하여 확인해 보기

### **배포된 Docker image 를 Docker Hub 로 올리기**

- Credentials 생성
    - Kind : Username with password
    - Username : docker hub 아이디
    - password : docker hub access key
    - ID : docker-hub / Description : docker-hub
- Jenkinsfile 작성

```yaml
pipeline {
	agent any
	parameters {
		choice(name: 'VERSION', choices: ['1.1.0','1.2.0','1.3.0'], description: '')
		booleanParam(name: 'executeTests', defaultValue: true, description: '')
	}
	stages {
		stage("init") {
			steps {
				script {
					gv = load "script.groovy"
				}
			}
		}
		stage("Checkout") {
			steps {
				checkout scm
			}
		}
		stage("Build") {
			steps {
				sh 'docker-compose build web'
			}
		}
		stage("test") {
			when {
				expression {
					params.executeTests
				}
			}
			steps {
				script {
					gv.testApp()
				}
			}
		}
		stage("Tag and Push") {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
				credentialsId: 'docker-hub', 
				usernameVariable: 'DOCKER_USER_ID', 
				passwordVariable: 'DOCKER_USER_PASSWORD'
				]]) {
					sh "docker tag jenkins-pipeline_web:latest ${DOCKER_USER_ID}/jenkins-app:${BUILD_NUMBER}"
					sh "docker login -u ${DOCKER_USER_ID} -p ${DOCKER_USER_PASSWORD}"
					sh "docker push ${DOCKER_USER_ID}/jenkins-app:${BUILD_NUMBER}"
				}
			}
		}
		stage("deploy") {
			steps {
				sh "docker-compose up -d"
			}
		}
	}
}
```