---
layout: single
title: Azure MLOps 실습자료
date: 2022-05-06 17:25:00 +0900
last_modified_at: 2022-05-06 17:25:00 +0900
category: mlops
tags: ["Open Cloud"]
published: false

---

## Azure DevOps 환경 생성

- **계정 생성**
    - 윈도우 Microsoft 계정이 있을 경우 연동됨
    - [https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/) 접속
        - 12개월 동안 인기 서비스 무료
        - 25개의 기타 서비스들 항상 무료
        - 30일 간 $200 크레딧 제공
    - [Agreement]
    - [Identify verification by phone]
    - [Identify verification by card]
        - 준비물 : VISA/AMERICAN EXPRESS/MASTER 신용카드 중 1개

- **인프라 배포**
    - Azure Portal 접속
        - [https://portal.azure.com/](https://portal.azure.com/)
    - 리소스 그룹 만들기
        - 리소스 그룹 : azure-mlops
        - 영역 : 미국 동부
        - 태그
            - 이름 : displayName
            - 값 : MLOps
    
- **Azure CLI 리눅스에 설치**
    - `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash`
    - 발생 가능한 에러
        - 증상
            
            Could not get lock /var/lib/dpkg/lock. It is held by process 743462 (unattended-upgr)
            
        - 해결
            
            ```bash
            sudo killall apt apt-get
            sudo rm /var/lib/apt/lists/lock
            sudo rm /var/cache/apt/archives/lock
            sudo rm /var/lib/dpkg/lock*
            sudo dpkg --configure -a
            sudo apt update
            ```
            
        

## Github Actions 를 이용한 **FastAPI App 자동 배포**

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a6e3857-fefc-4798-9d82-b06cad30c83f/Untitled.png) -->
![Untitled](/assets/img/mlops_opencloud1_1.png)


- 실습 Github 복사
    - Fork 또는 Local 복사 후 Github push
        
        [https://github.com/yunjjun/azure-webapp.git](https://github.com/yunjjun/azure-webapp.git)
        
        - [Red Wine Quality](https://www.kaggle.com/uciml/red-wine-quality-cortez-et-al-2009) 데이터
        - winequality-red.csv 파일 추가 후 push
    - azure-webapp
    - VM 리눅스에 Clone
        
        ```bash
        git clone -b master https://github.com/yunjjun/azure-webapp.git
        cd azure-webapp
        ```
        
- Docker contianer 생성 테스트
    - `docker-compose build web`
    - `docker-compose up -d`

- **Azure Container Registry 만들기**
    - 리소스 만들기
        - 종류 : Container Registry
        - 레지스트리 이름 : mlopscd
        - 위치 : 미국 동부
        - [SKU](https://docs.microsoft.com/ko-kr/partner-center/develop/product-resources#sku) : 기본
    - 액세스 키 얻기
        - mlops | 액세스 키
            - 관리 사용자 사용으로 변경
    - Azure CLI 로그인
        
        ```bash
        az login
        az acr login --name mlopscd
        ```
        
    - Docker 빌드하고 배포해 보기
        
        ```bash
        docker build . -t mlopscd.azurecr.io/mlops-cd:1.0
        docker images
        docker push mlopscd.azurecr.io/mlops-cd:1.0
        ```
        
    - Docker 이미지 확인
        - mlops | 리포지토리
        
- **Azure App Service 만들기**
    - 웹 앱 생성
        - 기본
            - 이름 : mlops-cd
            - 게시 : Docker 컨테이너
            - 운영 체제 : Linux
            - 지역 : East US
        - Docker
            - 이미지 소스 : Azure Container Registry
        - 확인
            
            [https://mlops-cd.azurewebsites.net/docs](https://mlops-cd.azurewebsites.net/docs)
            
    
- **Github Actions 설정**
    - prod.workflow.yml 작성
        - .github/workflows/prod.workflow.yml 생성
            
            ```yaml
            name: Build and deploy to production
            
            on: 
              push:
                branches: 
                  - master
                  
            jobs:
              build-and-deploy:
                runs-on: ubuntu-latest
                
                steps:
                
                - name: Checkout GitHub Actions
                  uses: actions/checkout@main
                  
                  
                - name: Login via Azure CLI
                  uses: azure/login@v1
                  with:
                    creds: ${{ secrets.AZURE_CREDENTIALS }}
                    
                    
                - name: Login to Azure Container Registry
                  uses: azure/docker-login@v1
                  with:
                    login-server: mlopscd.azurecr.io
                    username: ${{ secrets.REGISTRY_USERNAME }}
                    password: ${{ secrets.REGISTRY_PASSWORD }}
                 
                 
                - name: Build and push container image to registry 
                  run: |
                    docker build . -t mlopscd.azurecr.io/mlops-cd:${{ github.sha }}
                    docker push mlopscd.azurecr.io/mlops-cd:${{ github.sha }}
                    
                    
                - name: Deploy to App Service
                  uses: azure/webapps-deploy@v2
                  with:
                    app-name: 'mlops-cd'
                    images: 'mlopscd.azurecr.io/mlops-cd:${{ github.sha }}'
                    slot-name: 'staging'
            ```
            
        - Azure 정보
            
            `az ad sp create-for-rbac --name "github-actions" --role contributor --scopes /subscriptions/<GUID>/resourceGroups/azure-mlops --sdk-auth`
            
            - GUID
                - [리소스 그룹]-[구독 ID]
            - 결과
                
                ```bash
                {
                  "clientId": "<your client id>",
                  "clientSecret": "<your client secret>",
                  "subscriptionId": "<your subscription id>",
                  "tenantId": "<your tenant id>",
                  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
                  "resourceManagerEndpointUrl": "https://management.azure.com/",
                  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
                  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
                  "galleryEndpointUrl": "https://gallery.azure.com/",
                  "managementEndpointUrl": "https://management.core.windows.net/"
                }
                ```
                
        - Github Secrets 에 입력
            - AZURE_CREDENTIALS: 결과 전체 복사
            - REGISTRY_USERNAME: clientId
            - REGISTRY_PASSWORD: clientSecret
    - mlops-cd 배포센터 설정
        - 소스 : Github 작업
        - 조직-리포지토리-분기 선택
        - 사용 가능한 워크플로 사용
        - [저장]
    - 배포 슬롯 : 프로덕션 - 추가 옵션 보기 - S1 적용
        - 슬롯 추가
            - 이름 : staging
        - 배포 센터 설정
    - Git commit 테스트
        - Github Actions 동작 확인
    - staging ↔ production 교환 해주기
- 접속 테스트
    - [https://mlops-cd.azurewebsites.net/docs](https://mlops-cd.azurewebsites.net/docs)
    

## Azure ML 파이프라인 기초 실습

### **Microsoft Azure Machine Learning Studio**

- 기계 학습 리소스 생성
    - 기계 학습 작업 영역 만들기
        - 이름 : mlops-pipeline
        - 지역 : 미국 동부
        - 검토+만들기
- Studio 시작하기
    - Compute - Compute instances - [+New]
        - Standard_DS3_v2 선택
    - Notebooks 시작
- 데이터 업로드
    - data 폴더 생성 → winequality-red.csv 업로드
- 모델 학습
    - train.ipynb 생성 (파일 경로 수정 필요)
        
        ```python
        import logging
        from pathlib import Path
        
        import pandas as pd
        from joblib import dump
        from sklearn import preprocessing
        from sklearn.experimental import enable_hist_gradient_boosting  # noqa
        from sklearn.ensemble import HistGradientBoostingRegressor, RandomForestRegressor
        from sklearn.metrics import mean_squared_error
        from sklearn.model_selection import train_test_split
        
        logger = logging.getLogger(__name__)
        
        def prepare_dataset(test_size=0.2, random_seed=1):
            dataset = pd.read_csv(
                "./data/winequality-red.csv",
                delimiter=",",
            )
            dataset = dataset.rename(columns=lambda x: x.lower().replace(" ", "_"))
            train_df, test_df = train_test_split(dataset, test_size=test_size, random_state=random_seed)
            return {"train": train_df, "test": test_df}
        
        def train():
            logger.info("Preparing dataset...")
            dataset = prepare_dataset()
            train_df = dataset["train"]
            test_df = dataset["test"]
        
            # separate features from target
            y_train = train_df["quality"]
            X_train = train_df.drop("quality", axis=1)
            y_test = test_df["quality"]
            X_test = test_df.drop("quality", axis=1)
        
            logger.info("Training model...")
            scaler = preprocessing.StandardScaler().fit(X_train)
            X_train = scaler.transform(X_train)
            X_test = scaler.transform(X_test)
            model = RandomForestRegressor(max_depth=30).fit(X_train, y_train)
            # model = HistGradientBoostingRegressor(max_iter=50).fit(X_train, y_train)
        
            y_pred = model.predict(X_test)
            error = mean_squared_error(y_test, y_pred)
            logger.info(f"Test MSE: {error}")
        
            logger.info("Saving artifacts...")
            Path("artifacts").mkdir(exist_ok=True)
            dump(model, "artifacts/wine_model.joblib")
            dump(scaler, "artifacts/wine_scaler.joblib")
        
        if __name__ == "__main__":
            logging.basicConfig(level=logging.INFO)
            train()
        ```
        
- 모델 등록
    - register_model.ipynb 생성
        
        ```python
        from azureml.core import Workspace
        from azureml.core.model import Model
        
        ws = Workspace(subscription_id="<your subscription id>",
                       resource_group="azure-mlops",
                       workspace_name="mlops-pipeline")
        
        dname = "wine"
        model = Model.register(
            ws, 
            model_name=f"{dname}_model", 
            model_path=f"./artifacts/{dname}_model.joblib",
            tags={'area': f"{dname}", 'type': "regression"},
            description=f"RandomForest regression model to predict {dname} quality"
            )
        
        scaler = Model.register(
            ws, 
            model_name=f"{dname}_scaler", 
            model_path=f"./artifacts/{dname}_scaler.joblib")
        
        print(f"wine_model - name: {model.name}, id: {model.id}, ver: {model.version}")
        print(f"wine_scaler - name: {scaler.name}, id: {scaler.id}, ver: {scaler.version}")
        ```
        
- 모델 로드 테스트
    - load_model.ipynb 생성
        
        ```python
        from azureml.core import Workspace
        from azureml.core.model import Model
        
        ws = Workspace(subscription_id="<your subscription id>",
                       resource_group="azure-mlops",
                       workspace_name="mlops-pipeline")
        wine_model = Model(
            ws, 
            'wine_model', 
            version=1)
        wine_scaler = Model(
            ws, 
            'wine_scaler', 
            version=1
        )
        ```
        

## Azure ML 파이프라인 예제 실습

- azure_ml_example.ipynb 만들기
- 작업공간 생성

```python
from azureml.core import Workspace
ws = Workspace.from_config()
print('Workspace name: ' + ws.name, 
      'Azure region: ' + ws.location, 
      'Subscription id: ' + ws.subscription_id, 
      'Resource group: ' + ws.resource_group, sep='\n')
```

- 실험공간 생성

```python
from azureml.core import Experiment
experiment = Experiment(workspace=ws, name="diabetes-experiment")
```

- 데이터 준비

```python
from azureml.opendatasets import Diabetes
from sklearn.model_selection import train_test_split

x_df = Diabetes.get_tabular_dataset().to_pandas_dataframe().dropna()
y_df = x_df.pop("Y")

X_train, X_test, y_train, y_test = train_test_split(x_df, y_df, test_size=0.2, random_state=66)

print(X_train)
```

- 모델 훈련하면서 로그 남기고 모델 파일 업로드

```python
from sklearn.linear_model import Ridge
from sklearn.metrics import mean_squared_error
from sklearn.externals import joblib
import math

alphas = [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]

for alpha in alphas:
    run = experiment.start_logging()
    run.log("alpha_value", alpha)

    model = Ridge(alpha=alpha)
    model.fit(X=X_train, y=y_train)
    y_pred = model.predict(X=X_test)
    rmse = math.sqrt(mean_squared_error(y_true=y_test, y_pred=y_pred))
    run.log("rmse", rmse)

    model_name = "model_alpha_" + str(alpha) + ".pkl"
    filename = "outputs/" + model_name

    joblib.dump(value=model, filename=filename)
    run.upload_file(name=model_name, path_or_stream=filename)
    run.complete()
    print(f"{alpha} exp completed")
```

- Studio 에서 실험 결과 확인 및 모델 다운로드

```python
experiment
```

- Best model 탐색 후 다운로드

```python
minimum_rmse_runid = None
minimum_rmse = None

for run in experiment.get_runs():
    run_metrics = run.get_metrics()
    run_details = run.get_details()
    # each logged metric becomes a key in this returned dict
    run_rmse = run_metrics["rmse"]
    run_id = run_details["runId"]
    
    if minimum_rmse is None:
        minimum_rmse = run_rmse
        minimum_rmse_runid = run_id
    else:
        if run_rmse < minimum_rmse:
            minimum_rmse = run_rmse
            minimum_rmse_runid = run_id

print("Best run_id: " + minimum_rmse_runid)
print("Best run_id rmse: " + str(minimum_rmse))
```

```python
from azureml.core import Run
best_run = Run(experiment=experiment, run_id=minimum_rmse_runid)
print(best_run.get_file_names())
```

```python
best_run.download_file(name=str(best_run.get_file_names()[0]))
```

- DataStore 에 Input/Output 데이터셋 등록

```python
import numpy as np
from azureml.core import Dataset

np.savetxt('features.csv', X_train, delimiter=',')
np.savetxt('labels.csv', y_train, delimiter=',')

datastore = ws.get_default_datastore()
datastore.upload_files(files=['./features.csv', './labels.csv'],
                       target_path='diabetes-experiment/',
                       overwrite=True)

input_dataset = Dataset.Tabular.from_delimited_files(path=[(datastore, 'diabetes-experiment/features.csv')])
output_dataset = Dataset.Tabular.from_delimited_files(path=[(datastore, 'diabetes-experiment/labels.csv')])
```

- Best model 등록

```python
import sklearn

from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = Model.register(workspace=ws,
                       model_name='diabetes-experiment-model',
                       model_path=f"./{str(best_run.get_file_names()[0])}", 
                       model_framework=Model.Framework.SCIKITLEARN,  
                       model_framework_version=sklearn.__version__,  
                       sample_input_dataset=input_dataset,
                       sample_output_dataset=output_dataset,
                       resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5),
                       description='Ridge regression model to predict diabetes progression.',
                       tags={'area': 'diabetes', 'type': 'regression'})

print('Name:', model.name)
print('Version:', model.version)
```

- 모델 배포

```python
service_name = 'diabetes-service'

service = Model.deploy(ws, service_name, [model], overwrite=True)
service.wait_for_deployment(show_output=True)
```

- 배포 서비스 테스트
    - 노트북
        
        ```python
        import json
        
        input_payload = json.dumps({
            'data': X_train[0:2].values.tolist(),
            'method': 'predict'
        })
        
        output = service.run(input_payload)
        
        print(output)
        ```
        
    - [Models]-[Endpoints]-[서비스명]-[Test]
- 서비스 삭제

```python
service.delete()
```