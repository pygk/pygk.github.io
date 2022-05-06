---
layout: single
title: 실습 1. ML 모델 성능 Monitoring
date: 2022-05-06 17:04:00 +0900
last_modified_at: 2022-05-06 17:04:00 +0900
category: mlops
tags: ["Pipeline Monitoring"]
published: false

---

### FastAPI Serving API 생성

- 수행 절차
    - 완성 모습
        
        <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a107ade0-8ffa-49b1-ace2-ee0d4b3cba51/Untitled.png) -->
        ![Untitled](/assets/img/mlops_piplinemonitoring1_1.png)
        
    - 'js-fastapi-monitoring' 으로 폴더 생성
    - 기본 파일들 생성
        - requirements.txt, Dockerfile
            
            [requirements.txt]
            
            ```python
            fastapi==0.65.2
            numpy==1.18.5
            pandas==0.23.3
            prometheus-fastapi-instrumentator==5.7.0
            pydantic==1.6.2
            scikit-learn==0.24.0
            uvicorn==0.13.2
            joblib
            ```
            
            [Dockerfile]
            
            ```docker
            FROM python:3.7
            COPY requirements.txt requirements.txt 
            RUN pip install -r requirements.txt
            WORKDIR /js-fastapi-monitoring
            COPY . /js-fastapi-monitoring
            EXPOSE 80
            CMD ["uvicorn", "app.api:app", "--host", "0.0.0.0", "--port", "80"]
            ```
            
    - 훈련 코드 생성
        
        [train.py]
        
        - 패키지 로드, logging
            
            ```python
            import logging
            from pathlib import Path
            
            import pandas as pd
            from joblib import dump
            from sklearn import preprocessing
            from sklearn.experimental import enable_hist_gradient_boosting  # noqa
            from sklearn.ensemble import HistGradientBoostingRegressor
            from sklearn.metrics import mean_squared_error
            from sklearn.model_selection import train_test_split
            
            logger = logging.getLogger(__name__)
            ```
            
        - prepare_dataset 함수
            - [Red Wine Quality](https://www.kaggle.com/uciml/red-wine-quality-cortez-et-al-2009) 데이터 소개
            
            ```python
            def prepare_dataset(test_size=0.2, random_seed=1):
                dataset = pd.read_csv(
                    "winequality-red.csv",
                    delimiter=",",
                )
                dataset = dataset.rename(columns=lambda x: x.lower().replace(" ", "_"))
                train_df, test_df = train_test_split(dataset, test_size=test_size, random_state=random_seed)
                return {"train": train_df, "test": test_df}
            ```
            
        - train 함수
            - StandardScaler 로 표준화
            - HistGradientBoostingRegressor 사용 (max_iter=50)
            - model 과 scaler 는 [joblib artifact](https://joblib.readthedocs.io/en/latest/) 로 저장하여 활용
            
            ```python
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
                model = HistGradientBoostingRegressor(max_iter=50).fit(X_train, y_train)
            
                y_pred = model.predict(X_test)
                error = mean_squared_error(y_test, y_pred)
                logger.info(f"Test MSE: {error}")
            
                logger.info("Saving artifacts...")
                Path("artifacts").mkdir(exist_ok=True)
                dump(model, "artifacts/model.joblib")
                dump(scaler, "artifacts/scaler.joblib")
            
            if __name__ == "__main__":
                logging.basicConfig(level=logging.INFO)
                train()
            ```
            
        - 실행해서 artifacts 저장
            
            <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9996bec4-3c1b-4748-a47e-e804850d0997/Untitled.png) -->
            ![Untitled](/assets/img/mlops_piplinemonitoring1_2.png)
            
    - app 구성
        - 'app' 폴더 생성
            
            [schemas.py]
            
            - Data Validation 을 위한 [pydantic schema](https://pydantic-docs.helpmanual.io/usage/schema/) 작성
                - pydantic : Type annotations 을 사용해 데이터 구문 분석 및 유효성 검사를 자동으로 해주고 오류 시 error 를 반환해주는 유용한 라이브러리
                
                ```python
                from pydantic import BaseModel, Field
                
                feature_names = [
                    "fixed_acidity",
                    "volatile_acidity",
                    "citric_acid",
                    "residual_sugar",
                    "chlorides",
                    "free_sulfur_dioxide",
                    "total_sulfur_dioxide",
                    "density",
                    "ph",
                    "sulphates",
                    "alcohol_pct_vol",
                ]
                
                ## ge : greater than or equal / le : less than or equal
                class Wine(BaseModel):
                    fixed_acidity: float = Field(
                        ..., ge=0, description="grams per cubic decimeter of tartaric acid"
                    )
                    volatile_acidity: float = Field(
                        ..., ge=0, description="grams per cubic decimeter of acetic acid"
                    )
                    citric_acid: float = Field(..., ge=0, description="grams per cubic decimeter of citric acid")
                    residual_sugar: float = Field(
                        ..., ge=0, description="grams per cubic decimeter of residual sugar"
                    )
                    chlorides: float = Field(..., ge=0, description="grams per cubic decimeter of sodium chloride")
                    free_sulfur_dioxide: float = Field(
                        ..., ge=0, description="milligrams per cubic decimeter of free sulfur dioxide"
                    )
                    total_sulfur_dioxide: float = Field(
                        ..., ge=0, description="milligrams per cubic decimeter of total sulfur dioxide"
                    )
                    density: float = Field(..., ge=0, description="grams per cubic meter")
                    ph: float = Field(..., ge=0, lt=14, description="measure of the acidity or basicity")
                    sulphates: float = Field(
                        ..., ge=0, description="grams per cubic decimeter of potassium sulphate"
                    )
                    alcohol_pct_vol: float = Field(..., ge=0, le=100, description="alcohol percent by volume")
                
                class Rating(BaseModel):
                    quality: float = Field(
                        ...,
                        ge=0,
                        le=10,
                        description="wine quality grade ranging from 0 (very bad) to 10 (excellent)",
                    )
                ```
                
        - api 생성
            
            [app/api.py]
            
            - 패키지, scaler, model 로드, 기본 api 생성
            
            ```python
            from pathlib import Path
            import numpy as np
            from fastapi import FastAPI, Response
            from joblib import load
            from .schemas import Wine, Rating, feature_names
            # from .monitoring import instrumentator
            
            ROOT_DIR = Path(__file__).parent.parent
            
            app = FastAPI()
            scaler = load(ROOT_DIR / "artifacts/scaler.joblib")
            model = load(ROOT_DIR / "artifacts/model.joblib")
            
            @app.get("/")
            def root():
                return "Wine Quality Ratings"
            ```
            
            - predict, healthcheck 함수
            
            ```python
            @app.post("/predict", response_model=Rating)
            def predict(response: Response, sample: Wine):
                sample_dict = sample.dict()
                features = np.array([sample_dict[f] for f in feature_names]).reshape(1, -1)
                features_scaled = scaler.transform(features)
                prediction = model.predict(features_scaled)[0]
                response.headers["X-model-score"] = str(prediction)
                return Rating(quality=prediction)
            
            @app.get("/healthcheck")
            def healthcheck():
                return {"status": "ok"}
            ```
            
    - Docker-compose 생성
        - docker-compose.yml
        
        ```docker
        version: "3"
        
        services:
          web:
            build: .
            container_name: js-fastapi-monitoring
            volumes:
              - .:/code
            ports:
              - "5000:80"
        ```
        
    - 서버에서 실행해 보기
        - Github Repo (name : js-fastapi-monitoring) 생성하여 업로드
        - 서버에서 Git Pull
            - `git fetch && git pull origin master`
        - `docker-compose build web`
        - `docker-compose up -d`
    - 웹 확인
        - [localhost:5000](http://localhost:5000) 접속
            
            <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/687bac0c-6379-4511-806e-8b25f96fecea/Untitled.png) -->
            ![Untitled](/assets/img/mlops_piplinemonitoring1_3.png)
            

### FastAPI-Prometheus Metric 수집

- 수행 절차
    - FastAPI-Prometheus 연동
        - [prometheus_fastapi_instrumentator](https://github.com/trallnag/prometheus-fastapi-instrumentator) 소개
        
        [app/api.py]
        
        ```python
        from .monitoring import instrumentator
        ```
        
        [app/monitoring.py]
        
        - 패키지 로드, NAMESPACE, SUBSYSTEM 생성
            
            ```python
            import os
            from typing import Callable
            
            import numpy as np
            from prometheus_client import Histogram
            from prometheus_fastapi_instrumentator import Instrumentator, metrics
            from prometheus_fastapi_instrumentator.metrics import Info
            
            NAMESPACE = os.environ.get("METRICS_NAMESPACE", "fastapi")
            SUBSYSTEM = os.environ.get("METRICS_SUBSYSTEM", "model")
            ```
            
        - instrumentator 생성
            - excluded_handlers : "/metrics"
            - env_var_name : "ENABLE_METRICS"
            - [참고 문서](https://trallnag.github.io/prometheus-fastapi-instrumentator/)
            
            ```python
            instrumentator = Instrumentator(
                should_group_status_codes=True,
                should_ignore_untemplated=True,
                should_respect_env_var=True,
                should_instrument_requests_inprogress=True,
                excluded_handlers=["/metrics"],
                env_var_name="ENABLE_METRICS",
                inprogress_name="fastapi_inprogress",
                inprogress_labels=True,
            )
            ## ENABLE_METRICS 가 true 인 Runtime 에서만 동작하게 됨 
            
            instrumentator.add(
                metrics.request_size(
                    should_include_handler=True,
                    should_include_method=True,
                    should_include_status=True,
                    metric_namespace=NAMESPACE,
                    metric_subsystem=SUBSYSTEM,
                )
            )
            instrumentator.add(
                metrics.response_size(
                    should_include_handler=True,
                    should_include_method=True,
                    should_include_status=True,
                    metric_namespace=NAMESPACE,
                    metric_subsystem=SUBSYSTEM,
                )
            )
            instrumentator.add(
                metrics.latency(
                    should_include_handler=True,
                    should_include_method=True,
                    should_include_status=True,
                    metric_namespace=NAMESPACE,
                    metric_subsystem=SUBSYSTEM,
                )
            )
            instrumentator.add(
                metrics.requests(
                    should_include_handler=True,
                    should_include_method=True,
                    should_include_status=True,
                    metric_namespace=NAMESPACE,
                    metric_subsystem=SUBSYSTEM,
                )
            )
            ```
            
        - 새로운 Metric 생성
            - prometheus_client 의 Histogram 활용
                - [Metric 유형 참고 사이트](https://promlabs.com/blog/2020/09/25/metric-types-in-prometheus-and-promql)  [from PromLabs]
                - bucket : 임의로 지정해 둔 측정 범위를 의미
                - Histogram : bucket 안에 있는 metric 의 빈도 측정
            - [info](https://github.com/trallnag/prometheus-fastapi-instrumentator/blob/02e807b76cd2692cf543e207fd07f926a69046d6/prometheus_fastapi_instrumentator/metrics.py#L25) object 에 instrumentation 에 필요한 모든 것이 있음
                - request, response, modified_handler, modified_status, modified_duration 등
            
            ```python
            def regression_model_output(
                metric_name: str = "regression_model_output",
                metric_doc: str = "Output value of regression model",
                metric_namespace: str = "",
                metric_subsystem: str = "",
                buckets=(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, float("inf")),
            ) -> Callable[[Info], None]:
                METRIC = Histogram(
                    metric_name,
                    metric_doc,
                    buckets=buckets,
                    namespace=metric_namespace,
                    subsystem=metric_subsystem,
                )
            
                def instrumentation(info: Info) -> None:
                    if info.modified_handler == "/predict":
                        predicted_quality = info.response.headers.get("X-model-score")
                        if predicted_quality:
                            METRIC.observe(float(predicted_quality))
            
                return instrumentation
            
            buckets = (*np.arange(0, 10.5, 0.5).tolist(), float("inf"))
            instrumentator.add(
                regression_model_output(metric_namespace=NAMESPACE, metric_subsystem=SUBSYSTEM, buckets=buckets)
            )
            ```
            
        - api 에 연동 부분 추가
            - [expose](https://github.com/trallnag/prometheus-fastapi-instrumentator/blob/02e807b76cd2692cf543e207fd07f926a69046d6/prometheus_fastapi_instrumentator/instrumentation.py#L206)
            
            ```python
            from .monitoring import instrumentator
            ...
            instrumentator.instrument(app).expose(app, include_in_schema=False, should_gzip=True)
            ```
            
        - docker-compose.yml 내용 변경
            
            ```docker
            version: "3"
            
            services:
              web:
                build: .
                container_name: js-fastapi-monitoring
                volumes:
                  - .:/code
                ports:
                  - "5000:80"
                **environment:
                  - ENABLE_METRICS=true**
            ```
            
        - 로컬에서 git push
        - 서버에서 git pull
        - `docker-compose build web && docker-compose up -d`
        - [http://localhost:5000/metrics](http://localhost:5000/metrics) 확인

### Prometheus 와 Grafana 연동

- 수행 절차
    - prometheus 로컬 설치
        - prometheus.yml 작성 ([참고](https://prometheus.io/docs/prometheus/latest/configuration/configuration/))
            - global.scrape_interval : 몇 초마다 Metric 을 수집할 지 설정
            - global.evaluation_interval : AlertRule 을 어느 주기마다 evaluate 할 것인지 설정
                - evaluate 하게 되면 inactive, pending, firing 의 3가지 state 로 구분이 되고, 이를 기반으로 alertmanager에서의 알람 발송 여부를 정할 수 있다.
                - alertmanager의 rule은 expression을 사용해서 설정할 수 있다.
            - scrape_config
                - job_name : 메트릭을 수집해서 구분을 할 네이밍을 지정
                - static_configs.targets : Metric 을 수집할 서버 주소를 지정한다.
            
            ```yaml
            # prometheus.yml
            
            global:
              scrape_interval:     15s
              evaluation_interval: 30s
              # scrape_timeout is set to the global default (10s).
            
            scrape_configs:
            - job_name: fastapi
              honor_labels: true
              static_configs:
              - targets:
                - localhost:5000  # metrics from model
            ```
            
        - prometheus.yml 을 volume 으로 참조해서 실행
            - `docker run -d --name prom-docker --network=host -v <서버 경로>/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus`
        - [localhost:9090/targets](http://localhost:9090/targets) 에 접속하여 확인
            
            <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4674c84-70d7-4275-8877-c93176a5611a/Untitled.png) -->
            ![Untitled](/assets/img/mlops_piplinemonitoring1_4.png)
            
    - grafana 로컬 설치/ prometheus metric 연결 확인
        - `docker run -d --name grafana-docker --network=host grafana/grafana`
        - [localhost:3000](http://localhost:3000) 접속
            - 초기 아이디/비밀번호 : admin/admin
        - prometheus 연결 확인
            - [Configuration]-[Data sources]-[Add data source]-[Prometheus 선택]
                
                <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3647055e-ff70-470f-9d02-8804c6b97ab2/Untitled.png) -->
                ![Untitled](/assets/img/mlops_piplinemonitoring1_5.png)
                
            - Save & test 확인
            - Explore
            - New Panel
                - Data source : Prometheus
                - Metrics browser : fastapi_monitoring_regression_model_output_sum{}
                - Use Query

### Locust 를 이용한 Simulation 및 Dashboard 생성

- 수행 절차
    - [Locust](http://docs.locust.io/en/stable/what-is-locust.html) 란?
        - 구축된 서버의 성능을 측정하기 위한 스트레스 테스트 도구
        - 특징
            - @task 데코레이터의 인자값을 이용해 어느 task 를 더 많이 실행할 지 정의
    - 코드 작성
        - locust 폴더, locustfile.py 생성
        - Feature 와 dataset 정의
            
            [locustfile.py]
            
            ```python
            from locust import HttpUser, task
            import pandas as pd
            import random
            
            feature_columns = {
                "fixed acidity": "fixed_acidity",
                "volatile acidity": "volatile_acidity",
                "citric acid": "citric_acid",
                "residual sugar": "residual_sugar",
                "chlorides": "chlorides",
                "free sulfur dioxide": "free_sulfur_dioxide",
                "total sulfur dioxide": "total_sulfur_dioxide",
                "density": "density",
                "pH": "ph",
                "sulphates": "sulphates",
                "alcohol": "alcohol_pct_vol",
            }
            dataset = (
                pd.read_csv(
                    "winequality-red.csv",
                    delimiter=",",
                )
                .rename(columns=feature_columns)
                .drop("quality", axis=1)
                .to_dict(orient="records")
            )
            ```
            
        - test class 생성
            
            ```python
            class WinePredictionUser(HttpUser):
                @task(1)
                def healthcheck(self):
                    self.client.get("/healthcheck")
            
                @task(10)
                def prediction(self):
                    record = random.choice(dataset).copy()
                    self.client.post("/predict", json=record)
            
                @task(2)
                def prediction_bad_value(self):
                    record = random.choice(dataset).copy()
                    corrupt_key = random.choice(list(record.keys()))
                    record[corrupt_key] = "bad data"
                    self.client.post("/predict", json=record)
            ```
            
        - locust 폴더에 winequality-red.csv 복사해서 넣기
        - Git 반영
        - 서버에서 locust 실행
            
            ```bash
            pip install locust==1.4.1
            pip install pandas==0.23.3
            docker pull origin master
            locust -f locust/locustfile.py --host http://127.0.0.1:5000
            ```
            
        - [localhost:8089](http://localhost:8089) 접속하여 확인
    - Grafana Dashboard 생성
        - [localhost:3000](http://localhost:3000) 에 접속
        - + → Import → Import via panel json 에 아래 json 입력하고 Load
            
            ```json
            {
              "annotations": {
                "list": [
                  {
                    "builtIn": 1,
                    "datasource": "-- Grafana --",
                    "enable": true,
                    "hide": true,
                    "iconColor": "rgba(0, 211, 255, 1)",
                    "name": "Annotations & Alerts",
                    "target": {
                      "limit": 100,
                      "matchAny": false,
                      "tags": [],
                      "type": "dashboard"
                    },
                    "type": "dashboard"
                  }
                ]
              },
              "editable": true,
              "fiscalYearStartMonth": 0,
              "gnetId": null,
              "graphTooltip": 0,
              "id": 3,
              "links": [],
              "liveNow": false,
              "panels": [
                {
                  "collapsed": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {},
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 1,
                    "w": 24,
                    "x": 0,
                    "y": 0
                  },
                  "id": 18,
                  "panels": [],
                  "title": "Model Metrics",
                  "type": "row"
                },
                {
                  "datasource": null,
                  "gridPos": {
                    "h": 9,
                    "w": 5,
                    "x": 0,
                    "y": 1
                  },
                  "id": 10,
                  "options": {
                    "content": "# Model Predictions\n\n와인 데이터를 이용한 ML 모델 성능 모니터링",
                    "mode": "markdown"
                  },
                  "pluginVersion": "8.2.5",
                  "timeFrom": null,
                  "timeShift": null,
                  "type": "text"
                },
                {
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "mappings": [],
                      "max": 10,
                      "min": 0,
                      "noValue": "No Data",
                      "thresholds": {
                        "mode": "absolute",
                        "steps": [
                          {
                            "color": "red",
                            "value": null
                          },
                          {
                            "color": "yellow",
                            "value": 1
                          },
                          {
                            "color": "green",
                            "value": 3
                          },
                          {
                            "color": "#EAB839",
                            "value": 7
                          },
                          {
                            "color": "red",
                            "value": 9
                          }
                        ]
                      }
                    },
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 9,
                    "w": 4,
                    "x": 5,
                    "y": 1
                  },
                  "id": 8,
                  "options": {
                    "orientation": "auto",
                    "reduceOptions": {
                      "calcs": [
                        "mean"
                      ],
                      "fields": "",
                      "values": false
                    },
                    "showThresholdLabels": false,
                    "showThresholdMarkers": true,
                    "text": {}
                  },
                  "pluginVersion": "8.2.5",
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_regression_model_output_sum[3m])) / sum(rate(fastapi_model_regression_model_output_count[3m]))",
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "timeFrom": null,
                  "timeShift": null,
                  "title": "Model Score (3m avg)",
                  "type": "gauge"
                },
                {
                  "cards": {
                    "cardPadding": null,
                    "cardRound": null
                  },
                  "color": {
                    "cardColor": "#F2495C",
                    "colorScale": "sqrt",
                    "colorScheme": "interpolateViridis",
                    "exponent": 0.5,
                    "mode": "spectrum"
                  },
                  "dataFormat": "tsbuckets",
                  "datasource": null,
                  "description": "Average per second count of predictions which fall into each bucket.",
                  "gridPos": {
                    "h": 9,
                    "w": 15,
                    "x": 9,
                    "y": 1
                  },
                  "heatmap": {},
                  "hideZeroBuckets": true,
                  "highlightCards": true,
                  "id": 6,
                  "legend": {
                    "show": true
                  },
                  "pluginVersion": "7.2.1",
                  "reverseYBuckets": false,
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_regression_model_output_bucket[1m])) by (le)",
                      "format": "heatmap",
                      "interval": "",
                      "legendFormat": "{{le}}",
                      "refId": "A"
                    }
                  ],
                  "timeFrom": null,
                  "timeShift": null,
                  "title": "Model Prediction Distribution",
                  "tooltip": {
                    "show": true,
                    "showHistogram": false
                  },
                  "type": "heatmap",
                  "xAxis": {
                    "show": true
                  },
                  "xBucketNumber": null,
                  "xBucketSize": null,
                  "yAxis": {
                    "decimals": null,
                    "format": "short",
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true,
                    "splitFactor": null
                  },
                  "yBucketBound": "auto",
                  "yBucketNumber": null,
                  "yBucketSize": null
                },
                {
                  "collapsed": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {},
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 1,
                    "w": 24,
                    "x": 0,
                    "y": 10
                  },
                  "id": 16,
                  "panels": [],
                  "title": "Service Metrics",
                  "type": "row"
                },
                {
                  "datasource": null,
                  "gridPos": {
                    "h": 9,
                    "w": 5,
                    "x": 0,
                    "y": 11
                  },
                  "id": 12,
                  "options": {
                    "content": "# Request Throughput\n\n전체 Reqeust 중 5xx 에러로 돌아오지 않는 비율",
                    "mode": "markdown"
                  },
                  "pluginVersion": "8.2.5",
                  "timeFrom": null,
                  "timeShift": null,
                  "type": "text"
                },
                {
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "mappings": [],
                      "max": 1,
                      "min": 0.8,
                      "noValue": "No Data",
                      "thresholds": {
                        "mode": "absolute",
                        "steps": [
                          {
                            "color": "red",
                            "value": null
                          },
                          {
                            "color": "yellow",
                            "value": 0.95
                          },
                          {
                            "color": "green",
                            "value": 0.99
                          }
                        ]
                      },
                      "unit": "percentunit"
                    },
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 9,
                    "w": 4,
                    "x": 5,
                    "y": 11
                  },
                  "id": 14,
                  "options": {
                    "orientation": "auto",
                    "reduceOptions": {
                      "calcs": [
                        "mean"
                      ],
                      "fields": "",
                      "values": false
                    },
                    "showThresholdLabels": false,
                    "showThresholdMarkers": true,
                    "text": {}
                  },
                  "pluginVersion": "8.2.5",
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_http_requests_total{status!=\"5xx\"}[3m])) / sum(rate(fastapi_model_http_requests_total[3m]))",
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "timeFrom": null,
                  "timeShift": null,
                  "title": "HTTP Success Rate (3m avg)",
                  "type": "gauge"
                },
                {
                  "aliasColors": {},
                  "bars": false,
                  "dashLength": 10,
                  "dashes": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "unit": "reqps"
                    },
                    "overrides": []
                  },
                  "fill": 10,
                  "fillGradient": 0,
                  "gridPos": {
                    "h": 9,
                    "w": 15,
                    "x": 9,
                    "y": 11
                  },
                  "hiddenSeries": false,
                  "id": 2,
                  "legend": {
                    "alignAsTable": false,
                    "avg": false,
                    "current": false,
                    "hideEmpty": false,
                    "hideZero": false,
                    "max": false,
                    "min": false,
                    "show": true,
                    "total": false,
                    "values": false
                  },
                  "lines": true,
                  "linewidth": 1,
                  "nullPointMode": "null",
                  "options": {
                    "alertThreshold": true
                  },
                  "percentage": false,
                  "pluginVersion": "8.2.5",
                  "pointradius": 2,
                  "points": false,
                  "renderer": "flot",
                  "seriesOverrides": [],
                  "spaceLength": 10,
                  "stack": true,
                  "steppedLine": false,
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_http_requests_total[1m])) by (status) ",
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "thresholds": [],
                  "timeFrom": null,
                  "timeRegions": [],
                  "timeShift": null,
                  "title": "Requests",
                  "tooltip": {
                    "shared": true,
                    "sort": 0,
                    "value_type": "individual"
                  },
                  "type": "graph",
                  "xaxis": {
                    "buckets": null,
                    "mode": "time",
                    "name": null,
                    "show": true,
                    "values": []
                  },
                  "yaxes": [
                    {
                      "format": "reqps",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": "0",
                      "show": true
                    },
                    {
                      "format": "short",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": null,
                      "show": true
                    }
                  ],
                  "yaxis": {
                    "align": false,
                    "alignLevel": null
                  }
                },
                {
                  "datasource": null,
                  "description": "",
                  "gridPos": {
                    "h": 9,
                    "w": 5,
                    "x": 0,
                    "y": 20
                  },
                  "id": 20,
                  "options": {
                    "content": "# Request Latencies       ",
                    "mode": "markdown"
                  },
                  "pluginVersion": "8.2.5",
                  "timeFrom": null,
                  "timeShift": null,
                  "type": "text"
                },
                {
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "mappings": [],
                      "min": 0,
                      "thresholds": {
                        "mode": "absolute",
                        "steps": [
                          {
                            "color": "green",
                            "value": null
                          },
                          {
                            "color": "red",
                            "value": 80
                          }
                        ]
                      }
                    },
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 9,
                    "w": 4,
                    "x": 5,
                    "y": 20
                  },
                  "id": 23,
                  "options": {
                    "colorMode": "value",
                    "graphMode": "area",
                    "justifyMode": "auto",
                    "orientation": "auto",
                    "reduceOptions": {
                      "calcs": [
                        "last"
                      ],
                      "fields": "",
                      "values": false
                    },
                    "text": {},
                    "textMode": "auto"
                  },
                  "pluginVersion": "8.2.5",
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(fastapi_inprogress)",
                      "instant": false,
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "timeFrom": null,
                  "timeShift": null,
                  "title": "Requests In Progress",
                  "type": "stat"
                },
                {
                  "aliasColors": {},
                  "bars": false,
                  "dashLength": 10,
                  "dashes": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "unit": "s"
                    },
                    "overrides": []
                  },
                  "fill": 1,
                  "fillGradient": 0,
                  "gridPos": {
                    "h": 9,
                    "w": 15,
                    "x": 9,
                    "y": 20
                  },
                  "hiddenSeries": false,
                  "id": 4,
                  "legend": {
                    "avg": false,
                    "current": false,
                    "max": false,
                    "min": false,
                    "show": true,
                    "total": false,
                    "values": false
                  },
                  "lines": true,
                  "linewidth": 1,
                  "nullPointMode": "null",
                  "options": {
                    "alertThreshold": true
                  },
                  "percentage": false,
                  "pluginVersion": "8.2.5",
                  "pointradius": 2,
                  "points": false,
                  "renderer": "flot",
                  "seriesOverrides": [],
                  "spaceLength": 10,
                  "stack": false,
                  "steppedLine": false,
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "histogram_quantile(0.99, \n  sum(\n    rate(fastapi_model_http_request_duration_seconds_bucket[1m])\n  ) by (le)\n)",
                      "interval": "",
                      "legendFormat": "99th percentile",
                      "refId": "A"
                    },
                    {
                      "expr": "histogram_quantile(0.95, \n  sum(\n    rate(fastapi_http_request_duration_seconds_bucket[1m])\n  ) by (le)\n)",
                      "interval": "",
                      "legendFormat": "95th percentile",
                      "refId": "B"
                    },
                    {
                      "expr": "histogram_quantile(0.50, \n  sum(\n    rate(fastapi_http_request_duration_seconds_bucket[1m])\n  ) by (le)\n)",
                      "interval": "",
                      "legendFormat": "50th percentile",
                      "refId": "C"
                    }
                  ],
                  "thresholds": [],
                  "timeFrom": null,
                  "timeRegions": [],
                  "timeShift": null,
                  "title": "Latency",
                  "tooltip": {
                    "shared": true,
                    "sort": 0,
                    "value_type": "individual"
                  },
                  "type": "graph",
                  "xaxis": {
                    "buckets": null,
                    "mode": "time",
                    "name": null,
                    "show": true,
                    "values": []
                  },
                  "yaxes": [
                    {
                      "format": "s",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": null,
                      "show": true
                    },
                    {
                      "format": "short",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": null,
                      "show": true
                    }
                  ],
                  "yaxis": {
                    "align": false,
                    "alignLevel": null
                  }
                },
                {
                  "datasource": null,
                  "gridPos": {
                    "h": 9,
                    "w": 5,
                    "x": 0,
                    "y": 29
                  },
                  "id": 29,
                  "options": {
                    "content": "# Request Size       ",
                    "mode": "markdown"
                  },
                  "pluginVersion": "8.2.5",
                  "timeFrom": null,
                  "timeShift": null,
                  "type": "text"
                },
                {
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "mappings": [],
                      "noValue": "No Data",
                      "thresholds": {
                        "mode": "absolute",
                        "steps": [
                          {
                            "color": "green",
                            "value": null
                          },
                          {
                            "color": "#EAB839",
                            "value": 500
                          }
                        ]
                      },
                      "unit": "decbytes"
                    },
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 9,
                    "w": 4,
                    "x": 5,
                    "y": 29
                  },
                  "id": 31,
                  "options": {
                    "colorMode": "value",
                    "graphMode": "none",
                    "justifyMode": "auto",
                    "orientation": "auto",
                    "reduceOptions": {
                      "calcs": [
                        "mean"
                      ],
                      "fields": "",
                      "values": false
                    },
                    "text": {},
                    "textMode": "auto"
                  },
                  "pluginVersion": "8.2.5",
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_http_request_size_bytes_sum{handler=\"/predict\"}[3m])) / sum(rate(fastapi_model_http_request_size_bytes_count{handler=\"/predict\"}[3m]))",
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "timeFrom": null,
                  "timeShift": null,
                  "title": "Request Size (3m avg)",
                  "type": "stat"
                },
                {
                  "aliasColors": {},
                  "bars": false,
                  "dashLength": 10,
                  "dashes": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {
                      "unit": "decbytes"
                    },
                    "overrides": []
                  },
                  "fill": 1,
                  "fillGradient": 0,
                  "gridPos": {
                    "h": 9,
                    "w": 15,
                    "x": 9,
                    "y": 29
                  },
                  "hiddenSeries": false,
                  "id": 30,
                  "legend": {
                    "avg": false,
                    "current": false,
                    "max": false,
                    "min": false,
                    "show": true,
                    "total": false,
                    "values": false
                  },
                  "lines": true,
                  "linewidth": 1,
                  "nullPointMode": "null",
                  "options": {
                    "alertThreshold": true
                  },
                  "percentage": false,
                  "pluginVersion": "8.2.5",
                  "pointradius": 2,
                  "points": false,
                  "renderer": "flot",
                  "seriesOverrides": [],
                  "spaceLength": 10,
                  "stack": false,
                  "steppedLine": false,
                  "targets": [
                    {
                      "exemplar": true,
                      "expr": "sum(rate(fastapi_model_http_request_size_bytes_sum{handler=\"/predict\"}[1m])) / sum(rate(fastapi_model_http_request_size_bytes_count{handler=\"/predict\"}[1m]))",
                      "interval": "",
                      "legendFormat": "",
                      "refId": "A"
                    }
                  ],
                  "thresholds": [],
                  "timeFrom": null,
                  "timeRegions": [],
                  "timeShift": null,
                  "title": "Request Size",
                  "tooltip": {
                    "shared": true,
                    "sort": 0,
                    "value_type": "individual"
                  },
                  "type": "graph",
                  "xaxis": {
                    "buckets": null,
                    "mode": "time",
                    "name": null,
                    "show": true,
                    "values": []
                  },
                  "yaxes": [
                    {
                      "format": "decbytes",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": "0",
                      "show": true
                    },
                    {
                      "format": "short",
                      "label": null,
                      "logBase": 1,
                      "max": null,
                      "min": null,
                      "show": true
                    }
                  ],
                  "yaxis": {
                    "align": false,
                    "alignLevel": null
                  }
                },
                {
                  "collapsed": false,
                  "datasource": null,
                  "fieldConfig": {
                    "defaults": {},
                    "overrides": []
                  },
                  "gridPos": {
                    "h": 1,
                    "w": 24,
                    "x": 0,
                    "y": 38
                  },
                  "id": 25,
                  "panels": [],
                  "title": "Resource Metrics",
                  "type": "row"
                }
              ],
              "refresh": "5s",
              "schemaVersion": 32,
              "style": "dark",
              "tags": [],
              "templating": {
                "list": []
              },
              "time": {
                "from": "now-5m",
                "to": "now"
              },
              "timepicker": {},
              "timezone": "",
              "title": "Model Dashboard",
              "uid": "K1ZW-uxGz",
              "version": 2
            }
            ```
            
        - 결과 화면
            
            <!-- ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4932dd4-d390-45d2-b7cf-4b50ee051646/Untitled.png) -->
            ![Untitled](/assets/img/mlops_piplinemonitoring1_6.png)