---
layout: single
title: 실습 2. 기본 Jenkins CI Pipeline 생성 실습
date: 2022-05-06 16:56:00 +0900
last_modified_at: 2022-05-06 16:56:00 +0900
category: mlops
tags: ["Jenkins for MLOps"]
published: false

---

<aside>
💡 **실습 목표
1. Jenkinsfile 의 기본적인 구조를 알아보고 생성한다.
2. Jenkins 에서 Pipeline Job 을 생성하고 빌드한다.
3. post 를 이용해 모든 stage 가 실행된 후의 명령을 정의한다.
4. when 을 이용해 stage 가 실행되는 조건을 추가해 본다.
5. Jenkinsfile 환경변수를 설정해 본다.
6. parameters 를 이용하는 방법을 알아 본다.
7. 외부 groovy scripts 를 만들어 사용해 본다.
8. Replay 에서 수정 후 빌드 다시 실행해 보기**

</aside>

### **Jenkinsfile 의 기본적인 구조를 알아보고 생성한다.**

- 기본 코드 구조
    - pipeline : 반드시 맨 위에 있어야 한다.
    - agent : 어디에서 실행할 것인지 정의한다.
        - any, none, label, node, docker, dockerfile, kubernetes
        - agent 가 none 인 경우 stage 에 포함시켜야 함
            
            ```bash
            pipeline {
                agent none 
                stages {
                    stage('Example Build') {
                        agent { docker 'maven:3-alpine' }
                        steps {
                            echo 'Hello, Maven'
                            sh 'mvn --version'
                        }
                    }
                    stage('Example Test') {
                        agent { docker 'openjdk:8-jre' }
                        steps {
                            echo 'Hello, JDK'
                            sh 'java -version'
                        }
                    }
                }
            }
            ```
            
    - stages : 하나 이상의 stage 에 대한 모음
        - pipeline 블록 안에서 한 번만 실행 가능함
            
            ```yaml
            pipeline {
            	agent any
            	stages {
            		stage("build") {
            			steps {
            				echo 'building the applicaiton...'
            			}
            		}
            		stage("test") {
            			steps {
            				echo 'testing the applicaiton...'
            			}
            		}
            		stage("deploy") {
            			steps {
            				echo 'deploying the applicaiton...'
            			}
            		}
            	}
            }
            ```
            
- 본인 Github 에 새 Repository [ js-pipeline-project ] 생성
- Local 에서 'Jenkinsfile' 파일 생성하여 위 stages 코드 복사한 후 Github 업로드

### **Jenkins 에서 Pipeline Job 을 생성하고 빌드한다.**

- Pipeline 생성
    - 이름은 'jenkins-pipeline' 으로 입력
- Git 추가
    - [General]-[Branch Sources]-[Add source 선택]-[Git]
    - 위에서 생성한 Github Repository 추가
    - Credentials - [Add]
    - [Save] - Log 출력
- Pipeline Status 확인
    - 각 stage log 확인

### **post 를 이용해 모든 stage 가 실행된 후의 명령을 정의한다.**

- post 조건
    - always, changed, fixed, regression, aborted, success, unsuccessful, unstable, failure, notBuilt, cleanup
    
    ```yaml
    pipeline {
    	agent any
    	stages {
    		stage("build") {
    			steps {
    				echo 'building the applicaiton...'
    			}
    		}
    		stage("test") {
    			steps {
    				echo 'testing the applicaiton...'
    			}
    		}
    		stage("deploy") {
    			steps {
    				echo 'deploying the applicaiton...'
    			}
    		}
    	}
    	post {
    			always {
    				echo 'building..'
    			}
    			success {
    	            echo 'success'
    			}
    			failure {
    	            echo 'failure'
    			}
    		}
    	}
    ```
    

### **when 을 이용해 stage 가 실행되는 조건을 추가 해본다.**

- when 조건 추가
    - test stage 에서 Branch 이름에 따른 조건 추가
    - build stage 에서 Branch 이름에 따른 조건 추가
    
    ```bash
    pipeline {
    	agent any
    	stages {
    		stage("build") {
    			when {
    				expression {
    					env.GIT_BRANCH == 'origin/master'
    				}
    			}
    			steps {
    				echo 'building the applicaiton...'
    			}
    		}
    		stage("test") {
    			when {
    				expression {
    					env.GIT_BRANCH == 'origin/test' || env.GIT_BRANCH == ''
    				}
    			}
    			steps {
    				echo 'testing the applicaiton...'
    			}
    		}
    		stage("deploy") {
    			steps {
    				echo 'deploying the applicaiton...'
    			}
    		}
    	}
    }
    ```
    

### **Jenkinsfile 환경변수를 설정 해본다.**

- Jenkinsfile 자체 환경변수 목록 보기
    - [http://localhost:8080/env-vars.html/](http://localhost:8080/env-vars.html/) 접속 / 또는 [Jenkins pipeline-syntax](https://opensource.triology.de/jenkins/pipeline-syntax/globals) 참고
- Custom 환경변수 사용하기
    - echo 사용 시 큰 따옴표 주의
    
    ```yaml
    pipeline {
    	agent any
    	environment {
    		NEW_VERSION = '1.0.0'
    	}
    	stages {
    		stage("build") {
    			steps {
    				echo 'building the applicaiton...'
    				echo "building version ${NEW_VERSION}"
    			}
    		}
    		stage("test") {
    			steps {
    				echo 'testing the applicaiton...'
    			}
    		}
    		stage("deploy") {
    			steps {
    				echo 'deploying the applicaiton...'
    			}
    		}
    	}
    }
    ```
    
- Credentials 자격 증명 환경 변수로 사용하기
    - Jenkins credential 추가
        - [Jenkins 관리]-[Manage Credentials]-[Global credentials]-[Add credentials]
        - Username : admin_user / Password : 1234 / ID : admin_user_credentials
    - Jenkinsfile 에서 환경변수로 사용
        
        ```yaml
        pipeline {
        	agent any
        	environment {
        		NEW_VERSION = '1.0.0'
        		ADMIN_CREDENTIALS = credentials('admin_user_credentials')
        	}
        	stages {
        		stage("build") {
        			steps {
        				echo 'building the applicaiton...'
        				echo "building version ${NEW_VERSION}"
        			}
        		}
        		stage("test") {
        			steps {
        				echo 'testing the applicaiton...'
        			}
        		}
        		stage("deploy") {
        			steps {
        				echo 'deploying the applicaiton...'
        				echo "deploying with ${ADMIN_CREDENTIALS}"
        				sh 'printf ${ADMIN_CREDENTIALS}'
        			}
        		}
        	}
        }
        ```
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3fcf1717-9ca4-401c-b7b7-0138799cfb41/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins2_1.png)
        
    - Jenkins 플러그인 중 Credentials Plugin 확인
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae4e96e5-f2b5-4e0e-adf8-d0053b1cb3d2/Untitled.png) -->
        ![Untitled](/assets/img/mlops_jenkins2_2.png)
        
        ```yaml
        pipeline {
        	agent any
        	environment {
        		NEW_VERSION = '1.0.0'
        	}
        	stages {
        		stage("build") {
        			steps {
        				echo 'building the applicaiton...'
        				echo "building version ${NEW_VERSION}"
        			}
        		}
        		stage("test") {
        			steps {
        				echo 'testing the applicaiton...'
        			}
        		}
        		stage("deploy") {
        			steps {
        				echo 'deploying the applicaiton...'
        				withCredentials([[$class: 'UsernamePasswordMultiBinding',
        					credentialsId: 'admin_user_credentials', 
        					usernameVariable: 'USER', 
        					passwordVariable: 'PWD'
        				]]) {
        					sh 'printf ${USER}'
        				}
        			}
        		}
        	}
        }
        ```
        

### **parameters 를 이용하는 방법을 알아본다.**

- Jenkinsfile 에 parameter 추가
    
    ```yaml
    pipeline {
    	agent any
    	parameters {
    		string(name: 'VERSION', defaultValue: '', description: 'deployment version')
    		choice(name: 'VERSION', choices: ['1.1.0','1.2.0','1.3.0'], description: '')
    		booleanParam(name: 'executeTests', defaultValue: true, description: '')
    	}
    	stages {
    		stage("build") {
    			steps {
    				echo 'building the applicaiton...'
    			}
    		}
    		stage("test") {
    			steps {
    				echo 'testing the applicaiton...'
    			}
    		}
    		stage("deploy") {
    			steps {
    				echo 'deploying the applicaiton...'
    			}
    		}
    	}
    }
    ```
    
- executeTests 가 true 인 경우의 조건 추가해보기
    
    ```yaml
    pipeline {
    	agent any
    	parameters {
    		choice(name: 'VERSION', choices: ['1.1.0','1.2.0','1.3.0'], description: '')
    		booleanParam(name: 'executeTests', defaultValue: true, description: '')
    	}
    	stages {
    		stage("build") {
    			steps {
    				echo 'building the applicaiton...'
    			}
    		}
    		stage("test") {
    			when {
    				expression {
    					params.executeTests
    				}
    			}
    			steps {
    				echo 'testing the applicaiton...'
    			}
    		}
    		stage("deploy") {
    			steps {
    				echo 'deploying the applicaiton...'
    				echo "deploying version ${params.VERSION}"
    			}
    		}
    	}
    }
    ```
    
- 실제 Jenkinsfile 에 적용해 본다.
    - Build with Parameters 실행
        - 1.2.0 버전 선택
        - executeTests 선택 해제
    - test stage 를 건너뛰고 실행되는지 확인

### **외부 groovy scripts 를 만들어 사용해 본다.**

- groovy script 추가
    
    [script.groovy]
    
    ```groovy
    
    def buildApp() {
    	echo 'building the applications...'
    }
    
    def testApp() {
    	echo 'testing the applications...'
    }
    
    def deployApp() {
    	echo 'deploying the applicaiton...'
    	echo "deploying version ${params.VERSION}"
    }
    return this
    ```
    
    [Jenkinsfile]
    
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
    		stage("build") {
    			steps {
    				script {
    					gv.buildApp()
    				}
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
    				script {
    					gv.deployApp()
    				}
    			}
    		}
    	}
    }
    ```
    
    - Jenkinsfile 의 모든 환경변수는 groovy script 에서 사용 가능하다.
    - Github Repo 에 반영하고 실행/로그 확인
    - 빌드 결과 확인

### **Replay 에서 수정 후 빌드 다시 실행해 보기**

- testApp 에 echo 'Replay' 를 추가 후 다시 빌드
    
    <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01a91656-e29b-4254-9d4c-bb39374bdb1c/Untitled.png) -->
    ![Untitled](/assets/img/mlops_jenkins2_3.png)