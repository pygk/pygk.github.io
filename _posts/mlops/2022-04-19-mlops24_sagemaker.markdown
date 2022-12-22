---
layout: single
title: SageMaker Autopilot 실습
date: 2022-04-19 16:47:00 +0900
last_modified_at: 2022-04-19 16:47:00 +0900
category: mlops
tags: ["Amazon SageMaker"]
published: false
---

> 이번 강의에서는 **MNIST** Dataset 을 활용해, **multi-class Classification** 모델을 Amazon SageMaker **Autopilot** 으로 간단하게 만들어보겠습니다.
본 강의에서 SageMaker 를 사용함에 있어서 **사용량에 따라 aws 과금이 발생**할 수 있으니, 사용에 주의바랍니다. 과금이 발생하길 원하지 않는 수강생 분들은 실습을 진행하지 않으시는 것을 권장합니다.
> 

# 1. SageMaker 의 AutoML 기능 : Autopilot

> SageMaker 의 AutoML 기능인 autopilot 에 대해 알아보겠습니다.
> 

### AutoML 이란

- **데이터와 문제 그리고 평가 메트릭**이 정해져있을 때, 데이터 전처리 과정부터 적합한 모델 선택, HPO, 서빙까지 전 과정에서 **대부분의 작업을 자동화**하는 기술
    - Data Cleaning, Feature Engineering, HPO, Nueral Architecture Search 등을 포함한 ML 프로세스 전 과정의 자동화
    - 뒷단에 모든 작업을 감추고, 사용자에게는 super-high-level API 만 제공해서 뭔가 자동으로 모델을 생성해주는 것처럼 느끼게 하는 것
        - 단, 여기서의 모든 작업은 정말 모든 경우의 수를 parallel 하게 돌려서 최적의 성능을 내는 모델을 주는 것일 수도 있지만, 대부분의 경우에는 소수의 feature engineering / 소수의 model / 소수의 hp search space 내에서만 빠르게 수행
        - 하지만, SageMaker 의 autopilot 은 후보 model 의 종류가 많고, 리소스가 받쳐주기 때문에 엄청난 수의 작업을 병렬로 수행하여 더 높은 성능의 모델을 자동으로 생성
- **비전문가도** AI 를 활용하여 원하는 작업을 수행할 수 있도록 하기 위해 필요한 기술
    - 가장 수요가 많은 타겟 유저는 AI 에 대한 지식이 상대적으로 부족한 **End User**
- optuna, katib, autogluon, auto-sklearn 등의 저수준의 라이브러리가 존재하긴 하지만 아직 많이 부족
- Aws, GCP, Azure 3사에서도 모두 AutoML 기능을 제공

### Amazon SageMaker Autopilot 의 Workflow

- 2020년부터 서비스 제공, 관련 논문 발표
    - **[Amazon SageMaker Autopilot: a white box AutoML solution at scale](https://arxiv.org/abs/2012.08483)**
- 유저 관점
    - 1) Data 를 s3 에 업로드한다.
    - 2) SageMaker Studio 에서 autopilot experiment 를 create 할 때, ML problem type 과 target column 을 지정한다.
    - 3) autopilot experiment 가 완료되기까지 기다린다.
    - 4) auto-generated 된 Notebook 을 확인한다.
    - 5) model 을 deploy 하고 monitoring 한다.
- 시스템 관점
    
    <!-- ![**[Amazon SageMaker Autopilot: a white box AutoML solution at scale](https://arxiv.org/abs/2012.08483)**](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fbbe7145-d350-4771-99de-79570888889a/Untitled.png) -->
    ![Untitled](/assets/img/mlops_sagemaker_1.png)
    
    **[Amazon SageMaker Autopilot: a white box AutoML solution at scale](https://arxiv.org/abs/2012.08483)**
    
    - 1) data 를 train/valid split 하고, 분석한 뒤, 적합한 feature engineering 과정을 수행한다. 또한, 해당 과정에 대한 설명이 담긴 Jupyter Notebook 파일을 생성한다.
    - 2) pre-processed data 를 사용하여, ML problem type 에 맞는 model candidate 을 생성하고 각 model 을 학습한다.
    - 3) 적절한 model candidate 에 대해 HPO 과정을 수행한다.
    - 4) 각 모델의 performance 를 리더보드에 기록하고, 각 model 을 재현할 수 있는 Jupyter Notebook 파일을 생성한다.

### SageMaker Autopilot 의 장단점

- 장점
    - **WhiteBox**
        - 어떤 과정을 거쳐 AutoML 이 진행되었는지, 후보 모델로는 어떤 모델이 있었는지 등 전 과정에 대한 정보 제공
    - **Generated Notebook**
        - EDA 과정이 담긴 Jupyter Notebook 파일 제공
        - Model Candidate 과 그 사용법이 담긴 Jupyter Notebook 파일 제공
    - **Aws 의 다른 기능과의 연동**
        - s3 로 데이터 자동 저장 관리
        - SageMaker endpoint 로 원클릭 배포
        - CloudWatch 로 Monitoring 자동화
    - **높은 성능의 Baseline 모델** 제공
        - 다른 AutoML 툴과는 다르게, 상당히 많은 양의 모델을 검토하고 튜닝하기에 시간은 다소 오래 걸리지만 그만큼 좋은 성능을 가진 Baseline 모델을 제공
- 단점
    - **제한된 Data Type**
        - label 이 있는 tabular 형식의 정형 데이터에만 적합 (csv 로 표현)
    - **제한된 ML 문제 종류**
        - Classification(분류) or Regression(회귀)
    - **Pre-training 불가**
        - pretrained model 을 사용하거나, 도메인 지식을 첨가하여 feature engineering 을 미리 수행할 수는 없음

---

# 2. Quick Start

> MNIST dataset 을 활용한 multi-class classification model 을 SageMaker autopilot 을 사용해 만들어보겠습니다. 실습을 함께 진행하는 파트입니다. autopilot 전체 프로세스가 완료될 때까지 약 2 ~ 3 시간 정도 소요될 수 있습니다.
> 

### 1. 데이터 업로드

- kaggle 에서 mnist data csv 파일 받아서, s3 에 업로드합니다.
    - [https://www.kaggle.com/oddrationale/mnist-in-csv](https://www.kaggle.com/oddrationale/mnist-in-csv)
    - s3 에서 버킷을 새로운 이름으로 만들고, input 폴더를 생성한 뒤, 해당 폴더에 mnist_train.csv 파일과 mnist_test.csv 파일을 업로드합니다.
    - output 폴더도 만들어둡니다.

### 2. autopilot experiment 실행

- SageMaker Studio 에 접속합니다.
- Launcher 에서 New autopilot experiment 를 클릭합니다.
- experiment name 을 입력합니다.
- S3 bucket name 과 Dataset file name 을 위의 mnist dataset 경로로 지정합니다.
- Target column 을 `label` 로 지정합니다.
- Output data location 을 위의 S3 bucket 의 output 폴더 경로로 지정합니다.
- ML problem type 을 `Multiclass classification` 으로 지정합니다.
- Auto deploy 을 `off` 로 변경합니다.
- `Create Experiment` 버튼을 클릭합니다.
    - ***과금을 원하지 않으시는 분들은 `Do you want to run a complete experiment?` 에서 `No, run a pilot to create a notebook with candidate definitions` 를 체크해주시기 바랍니다. 혹은 `Create Experiment` 버튼을 클릭하지 않기 바랍니다.***
- 다음과 같은 화면이 나타나며, 정상적으로 autopilot experiment 가 실행되는 것을 확인합니다.

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c12ce4a1-826c-4ceb-9b9c-d2295f1a1f3b/Untitled.png) -->
![Untitled](/assets/img/mlops_sagemaker_2.png)

- 이제 약 2 ~ 3 시간 동안 총 250개의 model 을 검증하는 과정을 pre-processing → candidate definitions generated → feature engineering → model tuning 차례차례 수행

---

# 3. Autopilot 결과물 함께 살펴보기

> 몇 시간 후, autopilot experiment 가 완료되면 함께 autopilot 의 결과물인 Generated Notebook 을 살펴봅니다.
> 

### 중간 진행 상황 확인

- 약 20 분 정도 지나면, pre-processing 및 candidate definition generated 스텝이 완료되고 우측 상단에 Open candidate generation notebook, Open data exploration notebook 버튼이 생성됩니다.
    
    <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/52f87c6f-c919-4376-96f0-b90e6f32fa89/Untitled.png) -->
    ![Untitled](/assets/img/mlops_sagemaker_3.png)
    
- 진행 프로세스에서 생성된 결과물, 데이터는 output 경로로 입력한 s3 버킷에 저장
- Model Tuning 단계에 들어서면, 본격적으로 모든 Model 에 대해 HPO 과정을 거치는데 각 모델에 대한 진행 상황은 다음과 같이 확인 가능
    - experiments and trials 클릭하고, describe automl job 을 클릭하면
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc5e90f4-1824-4b2f-bde0-ff6379baad95/Untitled.png) -->
        ![Untitled](/assets/img/mlops_sagemaker_4.png)
        
    - 각 Trial 의 진행 상황 확인 가능
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef7b6f2e-5a89-427d-a49f-c4f02dd517fb/Untitled.png) -->
        ![Untitled](/assets/img/mlops_sagemaker_5.png)
        

### Data Exploration Notebook

- Generated EDA Notebook 샘플
    
    <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2fb9f160-25b0-4334-9da9-92d8a2aa467e/Untitled.png) -->
    ![Untitled](/assets/img/mlops_sagemaker_6.png)
    
- 1) **전체적인 흐름**에 대한 설명
    - input data 를 train/valid dataset 으로 split 을 했고, Data 의 size 는 어떻게 되고, 어떤 칼럼을 target 으로 사용했으며, multiclass 분류 문제를 풀었고...
- 2) **Dataset 샘플** 보여주기
    - 흔히 첫 단계로 Dataframe 의 head 를 보여주는 것과 동일
- 3) Dataset 에 대한 **기본적인 통계 분석**
    - column 별 data type
    - column 별 missing value 의 개수
        - missing value 가 너무 많으면 진짜 믿을 수 있는 데이터가 맞는지 확인해보라는 Suggestion comment 도 달림
    - column 별 unique value 의 개수
    - column 별 기본적인 statistics

### Candidate Definition Notebook

- Generated Candidate Definition Notebook 샘플
    
    <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cace3c2-5d3a-43c4-9383-9aac52dec2cd/Untitled.png) -->
    ![Untitled](/assets/img/mlops_sagemaker_7_.png)
    
- 1) **전체적인 흐름**에 대한 설명
    - 어떤 ML task 이고, 어떤 metric 을 maximize 하는 AutoML experiment 였는지...
- 2) SageMaker Jupyter notebook 에서 AutoML 의 결과물을 어떻게 사용해볼 수 있는지에 대한 **Setup 방법** 설명
- 3) **Generated Candidate Pipelines** (Feature selection + Model + HP) 에는 어떤 조합들이 있었는지, 어떤 타입의 리소스를 사용했는지에 대한 설명
- 4) 각 Candidate Pipeline 을 **어떻게 재현해볼 수 있는지**에 대한 설명
- 5) Best Model 을 어떻게 SageMaker Endpoint 로 **Deploy 할 수 있는지**에 대한 설명

---

# 4. SageMaker 자원 정리하기

- Amazon SageMaker 콘솔에서 모든 관련 자원을 정리합니다.
- 모든 사용자에 대해서 해당 사용자에서 실행 중인 모든 앱을 종료합니다.
    - 각 사용자 클릭
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e78271d-ee8f-4c67-a1b5-b7cab22a552f/Untitled.png) -->
        ![Untitled](/assets/img/mlops_sagemaker_8.png)
        
    - 모든 앱에 대해 `앱 삭제` 이후, 모든 앱의 상태가 Deleted 로 변경되면 `사용자 삭제`
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5bff69b2-a2a1-4f48-a01e-d11272f3026c/Untitled.png) -->
        ![Untitled](/assets/img/mlops_sagemaker_9.png)
        
- 모든 사용자가 삭제되면, 활성화된 `스튜디오 삭제` 버튼을 클릭하여 삭제를 진행합니다.
    
    <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4fa47bb9-ba77-4588-a440-e4eafa2e0c67/Untitled.png) -->
    ![Untitled](/assets/img/mlops_sagemaker_10.png)