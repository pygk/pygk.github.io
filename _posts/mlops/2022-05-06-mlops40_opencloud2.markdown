---
layout: single
title: GCP MLOps 실습자료
date: 2022-05-06 17:25:00 +0900
last_modified_at: 2022-05-06 17:25:00 +0900
category: mlops
tags: ["Open Cloud"]
published: false

---

## GCP DevOps 환경 생성

- 계정 생성
    - [https://cloud.google.com/](https://cloud.google.com/) 접속
    - [무료로 시작하기]
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9d49d29-ccf8-416d-9285-af33d5512639/Untitled.png) -->
        ![Untitled](/assets/img/mlops_opencloud2_1.png)
        

## Feast Feature Store 예제를 통한 GCP 익히기

- Colab 접속
    - [https://colab.research.google.com/](https://colab.research.google.com/)

### Feature Store 생성

- Feast [GCP] 설치
    
    ```python
    !pip install feast['gcp']
    !feast version
    ## 반드시 런타임 재시작
    ```
    
- Colab 인증
    
    ```python
    from google.colab import auth
    auth.authenticate_user()
    ```
    
- GCP 환경 설정
    
    > 반드시 실행해야 이후 에러가 발생하지 않음 [(관련 내용)](https://github.com/feast-dev/feast/issues/2117)
    > 
    > 
    > ```python
    > import dill
    > dill.extend(use_dill=False)
    > ```
    > 
    - Project 생성
        - [리소스 관리]-[Project 만들기]-[gcp-mlops-feast-project]
        - Project ID 확인
    - [**Cloud Firestore**](https://console.cloud.google.com/firestore/welcome)
        - 데이터 저장소 모드 선택
        - 데이터베이스 생성
        - 항목 만들기
            - 네임스페이스 : datastore
            - 종류 : mlops
    - **Cloud Storage** 에서 버킷 이름 확인
        - gcp-mlops-feast-project[.appspot.com](https://console.cloud.google.com/storage/browser/gcp-mlops-project.appspot.com;tab=objects?forceOnBucketsSortingFiltering=false&project=gcp-mlops-project)
    - 데이터 세트 이름 정하기
        - gcp_mlops_feast_dataset
    - 모델 이름 정하기
        - gcp_mlops_feast_model
    - 저장
        
        ```python
        PROJECT_ID= "<your project id>"
        BUCKET_NAME= "<your bucket name>"
        BIGQUERY_DATASET_NAME="<위에서 정한 데이터 세트 이름>"
        AI_PLATFORM_MODEL_NAME="<위에서 정한 모델 이름>"
        
        ! gcloud config set project $PROJECT_ID
        %env GOOGLE_CLOUD_PROJECT=$PROJECT_ID
        !echo project_id = $PROJECT_ID > ~/.bigqueryrc
        ```
        

- **BigQuery** 데이터 세트 만들기
    - 웹에서 만들기
        - [https://console.cloud.google.com/bigquery](https://console.cloud.google.com/bigquery) 에서 [데이터 추가]
    - 노트북에서 만들기
        
        ```python
        ! bq mk $BIGQUERY_DATASET_NAME
        ```
        
- Feast feature repository 초기화
    
    ```python
    ! feast init fraud_tutorial -t gcp
    %cd fraud_tutorial/
    ! ls
    ```
    
    - feature_store.yaml 수정
        
        ```python
        feature_store = \
        f"""project: fraud_tutorial
        registry: gs://{BUCKET_NAME}/registry.db
        provider: gcp"""
        
        with open('feature_store.yaml', "w") as feature_store_file:
            feature_store_file.write(feature_store)
        
        # Print our feature_store.yaml
        ! cat feature_store.yaml
        ```
        
        > project: fraud_tutorial
        registry: gs://<your bucket name>/registry.db
        provider: gcp
        > 
- Feast 배포
    
    ```python
    ! feast apply
    ```
    

### 새로운 Feature 생성

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ac64631-8d90-43d8-b588-7310ab20e487/Untitled.png) -->
![Untitled](/assets/img/mlops_opencloud2_2.png)

- 배포된 데이터 조회 해보기
    
    ```python
    %%bigquery
    
    select * from feast-oss.fraud_tutorial.transactions limit 1000
    ```
    
- BigQuery 활용 Feature Table 생성
    
    ```python
    from datetime import datetime, timedelta
    from google.cloud import bigquery
    import time
    
    def generate_user_count_features(aggregation_end_date):
        table_id  = f"{PROJECT_ID}.{BIGQUERY_DATASET_NAME}.user_count_transactions_7d"
    
        client = bigquery.Client()
        job_config = bigquery.QueryJobConfig(destination=table_id, write_disposition='WRITE_APPEND')
    
        aggregation_start_date = datetime.now() - timedelta(days=7)
    
        sql = f"""
        SELECT
            src_account AS user_id,
            COUNT(*) AS transaction_count_7d,
            timestamp'{aggregation_end_date.isoformat()}' AS feature_timestamp
        FROM
            feast-oss.fraud_tutorial.transactions
        WHERE
            timestamp BETWEEN TIMESTAMP('{aggregation_start_date.isoformat()}')
            AND TIMESTAMP('{aggregation_end_date.isoformat()}')
        GROUP BY
            user_id
        """
    
        query_job = client.query(sql, job_config=job_config)
        query_job.result()
        print(f"Generated features as of {aggregation_end_date.isoformat()}")
    
    def backfill_features(earliest_aggregation_end_date, interval, num_iterations):
        aggregation_end_date = earliest_aggregation_end_date
        for _ in range(num_iterations):
            generate_user_count_features(aggregation_end_date=aggregation_end_date)
            time.sleep(1)
            aggregation_end_date += interval
    
    if __name__ == '__main__':
        backfill_features(
            earliest_aggregation_end_date=datetime.now() - timedelta(days=7),
            interval=timedelta(days=1),
            num_iterations=8
        )
    ```
    
- 새로운 Feature 확인
    
    ```python
    %%bigquery 
    
    select * from <데이터 세트 이름>.user_count_transactions_7d limit 100
    ```
    
- FeatureView 생성
    - fraud_features.py 생성
        
        ```python
        fraud_features = \
        f"""
        from datetime import timedelta
        from feast import BigQuerySource, FeatureView, Entity, ValueType
        
        # Add an entity for users
        user_entity = Entity(
            name="user_id",
            description="A user that has executed a transaction or received a transaction",
            value_type=ValueType.STRING
        )
        
        # Add a FeatureView based on our new table
        driver_stats_fv = FeatureView(
            name="user_transaction_count_7d",
            entities=["user_id"],
            ttl=timedelta(weeks=1),
            batch_source=BigQuerySource(
                table_ref=f"{PROJECT_ID}.{BIGQUERY_DATASET_NAME}.user_count_transactions_7d",
                event_timestamp_column="feature_timestamp"))
        
        # Add two FeatureViews based on existing tables in BigQuery
        user_account_fv = FeatureView(
            name="user_account_features",
            entities=["user_id"],
            ttl=timedelta(weeks=52),
            batch_source=BigQuerySource(
                table_ref=f"feast-oss.fraud_tutorial.user_account_features",
                event_timestamp_column="feature_timestamp"))
        
        user_has_fraudulent_transactions_fv = FeatureView(
            name="user_has_fraudulent_transactions",
            entities=["user_id"],
            ttl=timedelta(weeks=52),
            batch_source=BigQuerySource(
                table_ref=f"feast-oss.fraud_tutorial.user_has_fraudulent_transactions",
                event_timestamp_column="feature_timestamp"))
        """
        
        with open('fraud_features.py', "w") as fraud_features_file:
            fraud_features_file.write(fraud_features)
        ```
        
    - 예제 파일인 driver_repo.py 삭제
- Feast 배포
    
    ```python
    !feast apply
    ```
    

### 모델 훈련과 배포

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b6f8c4eb-28d6-4c8f-b420-513ad8d6a82a/Untitled.png) -->
![Untitled](/assets/img/mlops_opencloud2_3.png)

- 훈련 데이터 추출
    
    ```python
    from datetime import datetime, timedelta
    from feast import FeatureStore
    
    store = FeatureStore(repo_path=".")
    
    now = datetime.now()
    two_days_ago = datetime.now() - timedelta(days=2)
    
    training_data = store.get_historical_features(
        entity_df=f"""
        select 
            src_account as user_id,
            timestamp,
            is_fraud
        from
            feast-oss.fraud_tutorial.transactions
        where
            timestamp between timestamp('{two_days_ago.isoformat()}') 
            and timestamp('{now.isoformat()}')""",
        features=[
            "user_transaction_count_7d:transaction_count_7d",
            "user_account_features:credit_score",
            "user_account_features:account_age_days",
            "user_account_features:user_has_2fa_installed",
            "user_has_fraudulent_transactions:user_has_fraudulent_transactions_7d"
        ],
        full_feature_names=True
    ).to_df()
    
    training_data.head()
    ```
    
- 모델 훈련
    
    ```python
    from sklearn.linear_model import LinearRegression
    
    training_data.dropna(inplace=True)
    
    X = training_data[[
        "user_transaction_count_7d__transaction_count_7d", 
        "user_account_features__credit_score",
        "user_account_features__account_age_days",
        "user_account_features__user_has_2fa_installed",
        "user_has_fraudulent_transactions__user_has_fraudulent_transactions_7d"
    ]]
    y = training_data["is_fraud"]
    
    model = LinearRegression()
    model.fit(X, y)
    ```
    
    ```python
    samples = X.iloc[:2]
    model.predict(samples)
    ```
    
- 모델 저장
    
    ```python
    import joblib
    
    joblib.dump(model, "model.joblib")
    ```
    
    - Bucket 으로 모델 업로드
        
        ```python
        !gsutil cp ./model.joblib gs://$BUCKET_NAME/model_dir/model.joblib
        ```
        
    - **Google AI-Platform** 에 모델 생성
        - 웹에서 생성
            - 모델 이름 : gcp_mlops_feast_model
            - 모델 버전 : version_1
            - 런타임 버전 : 2.2
            - Python 버전 : 3.7
            - Framework : sckikit-learn
            - 모델 파일 경로 : gs://<your bucket name>/model_dir
        - 노트북에서 생성
            
            ```python
            !gcloud ai-platform models create $AI_PLATFORM_MODEL_NAME --region global
            
            !gcloud ai-platform versions create version_1 \
            --model $AI_PLATFORM_MODEL_NAME \
            --runtime-version 2.2  \
            --python-version 3.7 \
            --framework scikit-learn \
            --origin gs://$BUCKET_NAME/model_dir \
            --region global \
            --verbosity "debug"
            ```
            
- 모델 Endpoint 테스트
    
    ```python
    import googleapiclient.discovery
    from IPython.display import clear_output
    
    instances = [
      [6.7, 3.1, 4.7, 1.5, 0],
      [4.6, 3.1, 1.5, 0.2, 0],
    ]
    
    service = googleapiclient.discovery.build('ml', 'v1')
    name = f'projects/{PROJECT_ID}/models/{AI_PLATFORM_MODEL_NAME}/'
    
    response = service.projects().predict(
        name=name,
        body={'instances': instances}
    ).execute()
    
    if 'error' in response:
        raise RuntimeError(response['error'])
    else:
      clear_output()
      print("Predictions: ", response['predictions'])
    ```
    

### Online Store 로 데이터 로드 (Materialize)

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5b58f8ab-345e-449d-937b-8f55f43b9144/Untitled.png) -->
![Untitled](/assets/img/mlops_opencloud2_4.png)

- 마지막 materialize 실행 시점으로부터 데이터 로드
    
    ```python
    !feast materialize-incremental $(date -u +"%Y-%m-%dT%H:%M:%S")
    ```
    

### 모델 추론

<!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3b987bda-b27f-4a2f-b8c2-1e92a7f11903/Untitled.png) -->
![Untitled](/assets/img/mlops_opencloud2_5.png)

```python
import googleapiclient.discovery

def predict(entity_rows):
    feature_vector = store.get_online_features(
        features=[
        "user_transaction_count_7d:transaction_count_7d",
        "user_account_features:credit_score",
        "user_account_features:account_age_days",
        "user_account_features:user_has_2fa_installed",
        "user_has_fraudulent_transactions:user_has_fraudulent_transactions_7d"
    ],
        entity_rows=entity_rows
    ).to_dict()
    
    del feature_vector["user_id"]

    instances = [
        [feature_values[i] for feature_values in feature_vector.values()]
        for i in range(len(entity_rows))
    ]
    
    service = googleapiclient.discovery.build('ml', 'v1')
    name = f'projects/{PROJECT_ID}/models/{AI_PLATFORM_MODEL_NAME}'
    
    response = service.projects().predict(
        name=name,
        body={'instances': instances}
    ).execute()
    
    clear_output()
    return response

predict)
```

### 작업 실행 예약하기

- **Cloud Build API**
    - [https://console.cloud.google.com/apis/library/cloudbuild.googleapis.com](https://console.cloud.google.com/apis/library/cloudbuild.googleapis.com?project=mlops-project-334512)
    - 사용 설정
- **Google Cloud Functions** 생성
    - cloud_function 폴더 생성
        
        ```python
        !mkdir cloud_function
        ```
        
    - cloud_function/configs.py 생성
        
        ```python
        configs = f"""
        project_id = '{PROJECT_ID}'
        bigquery_dataset_name = '{BIGQUERY_DATASET_NAME}'
        bucket_name = '{BUCKET_NAME}'
        """
        
        with open('cloud_function/configs.py', "w") as configs_file:
            configs_file.write(configs)
        ```
        
    - cloud_function/main.py
        
        ```python
        %%writefile cloud_function/main.py
        from datetime import datetime, timedelta
        from google.cloud import bigquery
        from feast import FeatureStore, RepoConfig
        from configs import project_id, bigquery_dataset_name, bucket_name
        
        def generate_user_count_features(aggregation_end_date):
            table_id  = f"{project_id}.{bigquery_dataset_name}.user_count_transactions_7d"
        
            client = bigquery.Client()
            job_config = bigquery.QueryJobConfig(destination=table_id, write_disposition='WRITE_APPEND')
        
            aggregation_start_date = datetime.now() - timedelta(days=7)
        
            sql = f"""
            SELECT
                src_account AS user_id,
                COUNT(*) AS transaction_count_7d,
                timestamp'{aggregation_end_date.isoformat()}' AS feature_timestamp
            FROM
                feast-oss.fraud_tutorial.transactions
            WHERE
                timestamp BETWEEN TIMESTAMP('{aggregation_start_date.isoformat()}')
                AND TIMESTAMP('{aggregation_end_date.isoformat()}')
            GROUP BY
                user_id
            """
        
            query_job = client.query(sql, job_config=job_config)
            query_job.result()
            print("Generated features as of: ", aggregation_end_date.isoformat())
        
        def materialize_features(end_date):
            store = FeatureStore(
                config=RepoConfig(
                    project="fraud_tutorial",
                    registry=f"gs://{bucket_name}/registry.db",
                    provider="gcp"
                )
            )
            store.materialize_incremental(end_date=datetime.now())
        
        def main(data, context):
            generate_user_count_features(aggregation_end_date=datetime.now())
            materialize_features(end_date=datetime.now())
        ```
        
    - cloud_function/requirements.txt 생성
        
        ```python
        %%writefile cloud_function/requirements.txt
        google-cloud-bigquery
        feast[gcp]
        ```
        
    - cloud function 실행
        
        ```python
        !cd cloud_function && gcloud functions deploy feast-update-features \
        --entry-point main \
        --runtime python37 \
        --trigger-resource feast-schedule \
        --trigger-event google.pubsub.topic.publish \
        --timeout 540s
        ```
        
- Google Cloud Functions 결과 확인
    - [https://console.cloud.google.com/functions/](https://console.cloud.google.com/functions/)
- 작업 예약하기
    - 웹에서 하기
        - [https://console.cloud.google.com/cloudscheduler](https://console.cloud.google.com/cloudscheduler)
    - 노트북에서 하기
        
        ```python
        !gcloud scheduler jobs create pubsub feast-daily-job \
        --schedule "0 22 * * *" \
        --topic feast-schedule \
        --message-body "This job schedules feature materialization once a day."
        ```
        
        - 웹에서 확인

### 모든 자원 해제하기

```python
!bq rm -t -f feast_fraud_tutorial.user_count_transactions_7d
!bq rm -r -f -d mlops-project
!gcloud functions delete feast-update-features
!gcloud scheduler jobs delete feast-daily-job --project $PROJECT_ID
```

## Feast FastAPI App 배포

### Project 설정

- IAM 및 관리자에서 자격 증명 얻기
    - 서비스 계정 - 작업 - 키 관리 -  키 추가 - 새 키 만들기

### Feast FastAPI App 생성

- 우측 상단 [Cloud shell 활성화] 클릭
- `git clone -b master https://github.com/yunjjun/gcp-feast-app.git`

```bash
cd gcp-feast-app
virtualenv env
source env/bin/activate
pip install -r requirements.txt
```

- [편집기 열기]
    - [env 폴더에 json key 파일 생성]
    - main.py
        - PROJECT *ID, BUCKET*NAME, BIGQUERY_DATASET_NAME,
            
            AI_PLATFORM_MODEL_NAME 변경
            
        - feature_store.yaml 내 registry 변경
        - JSON 파일 이름 변경
    - app.yaml
        
        ```bash
        runtime: python37
        entrypoint: gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
        instance_class: F2
        ```
        
- `gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app`
- [포트 변경 - 8000] - [변경 및 미리보기]
- 웹 확인

### GCP App 생성

- Cloud Build API 사용 설정
- `gcloud app create`
- 5번 선택
- `gcloud app deploy app.yaml`
- `gcloud app browse`
- target url 접속 테스트