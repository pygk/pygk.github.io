---
layout: single
title: 실습 1. Jenkins 설치
date: 2022-05-06 14:16:00 +0900
last_modified_at: 2022-05-06 14:16:00 +0900
category: mlops
tags: ["Jenkins for MLOps"]
published: false

---

- Jenkins 특징
    - Jenkinsfile
        - `Jenkinsfile` 을 이용해 Job 혹은 파이프라인을 정의할 수 있다. `Jenkinsfile` 덕분에 일반 소스코드를 다루는 Github 업로드, Vscode 로 수정하는 것으로 파일을 이용할 수 있음
        - 기본적으로 `Jenkinsfile`을 통해 젠킨스를 실행함
    - Scripted Pipeline (스크립트 파이프라인)
        - Jenkins 관련 구조를 자세히 가지지 않고 프로그램의 흐름을 Java 와 유사한 [Groovy](https://groovy-lang.org/) 라는 동적 객체 지향 프로그래밍 언어를 이용해 관리되었음
        - 매우 유연하지만 시작하기가 어려움
        
        ```groovy
        node { ## 빌드를 수행할 node 또는 agent를 의미한다.
            stage("Stage 1"){
                echo "Hello"
            }
            stage("Stage 2"){
                echo "World"
                sh "sleep 5"
            }
            stage("Stage 3"){
                echo "Good to see you!"
            }
        }
        ```
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac82a9bf-5b29-4bcf-b966-97d36f6c815d/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins0_1.png)
        
    - Declarative Pipeline (선언적 파이프라인)
        - 2016년 경 [Cloudbees 에서 개발](https://docs.cloudbees.com/docs/admin-resources/latest/pipeline-syntax-reference-guide/declarative-pipeline)
        - 사전에 정의된 구조만 사용할 수 있기 때문에 CI/CD 파이프라인이 단순한 경우에 적합하며 아직은 많은 제약사항이 따른다.
        - [공식 문서](https://www.jenkins.io/doc/book/pipeline/syntax/)
        
        ```bash
        pipeline {
            agent any
            stages {
                stage('Stage 1') {
                    steps {
                        script {
                            echo 'Hello'
                        }
                    }
                }
        
                stage('Stage 2') {
                    steps {
                        script {
                            echo 'World'
                            sh 'sleep 5'
                        }
                    }
                }
        
                stage('Stage 3') {
                    steps {
                        script {
                            echo 'Good to see you!'
                        }
                    }
                }
            }
        }
        ```
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/682141e3-72be-4c85-99f6-16d73462277e/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins0_2.png)

<aside>
💡 **실습 목표
1. Jenkins 를 Local 과 Docker 로 각각 설치한 후 기본 사용법을 익혀 본다.
2. 생성한 Docker Image 를 Docker Hub 에 Push 한다.**

</aside>

### **Jenkins 를 Local 과 Docker 로 각각 설치한 후 기본 사용법을 익혀 본다**

- Local 에 직접 설치해보기
    - JDK 설치
        
        ```bash
        sudo apt install openjdk-11-jre-headless
        ```
        
    - Key 다운로드
        
        ```bash
        wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
        echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
        ```
        
    - Jenkins 설치하기
        
        ```bash
        # sudo apt-get update
        sudo apt-get install jenkins
        ```
        
    - 정상여부 확인
        
        ```bash
        sudo systemctl status jenkins
        # 재시작 : sudo service jenkins restart
        ```
        
    - 초기 패스워드 확인
        
        ```bash
        sudo cat /var/lib/jenkins/secrets/initialAdminPassword
        ```
        
    - 브라우저 접속 (localhost:8080)
        - 초기 비밀번호 사용하여 로그인
    - 플러그인 설치
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6be3221a-d5a9-4bff-8d5d-d5fd83a581d4/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins1_1.png)
        
    - 계정 만들기
        - admin_user / 1234
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/997d81bc-1ded-44e1-b658-98576051f28c/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins1_2.png)