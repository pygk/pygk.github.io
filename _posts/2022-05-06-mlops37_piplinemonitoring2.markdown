---
layout: single
title: 실습 2. Jenkins 를 이용한 ML 모델 업데이트 및 자동 배포
date: 2022-05-06 17:18:00 +0900
last_modified_at: 2022-05-06 17:18:00 +0900
category: mlops
tags: ["Pipeline Monitoring"]
published: false

---

- Jenkins 재시작
    - `sudo service jenkins restart`
- Jenkins Pipeline 생성
    - [새로운 item] - [Pipeline : monitoring-pipeline]
    - Jenkins 설정
- Jenkinsfile 생성
    
    ```json
    pipeline {
    	agent any
    	stages {
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
    		stage("deploy") {
    			steps {
    				sh "docker-compose up -d"
    			}
    		}
    		stage("Update model") {
    			steps {
    				sh "docker exec -i js-fastapi-monitoring python train.py"
    			}
    		}
    	}
    }
    ```
    
- Jenkins 빌드 실행
- 'GitHub hook trigger for GITScm polling' 선택
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
- HistGradientBoostingRegressor 를 [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html#sklearn.ensemble.RandomForestRegressor) 로 변경해 보기
    - import 에 추가
        
        ```python
        from sklearn.ensemble import HistGradientBoostingRegressor, RandomForestRegressor
        ...
        model = RandomForestRegressor(max_depth=30).fit(X_train, y_train)
        ```
        
    - **max_depth=30**
    - Jenkins 자동 빌드 및 배포 확인
    - Granfana Dashboard 변화 확인