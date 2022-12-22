---
layout: single
title: 2. Prometheus & Grafana
date: 2022-04-19 15:04:00 +0900
last_modified_at: 2022-04-19 15:04:00 +0900
category: mlops
tags: ["Model Monitoring"]
published: false
---

# 1. Prometheus

[Prometheus](https://github.com/prometheus)

> Prometheus is a free software application used for event monitoring and alerting
- Wikipedia
> 
- 2012 년 SoundCloud 에서 만든 모니터링 & 알람 프로그램
- 2016 년 CNCF 에 Joined, 2018 년 Graduated 하여 완전 독립형 오픈소스 프로젝트로 발전
- 쿠버네티스에 종속적이지 않고, binary 혹은 docker container 형태로도 사용하고 배포 가능
- 쿠버네티스 생태계의 오픈소스 중에서는 사실상의 표준
    - 구조가 쿠버네티스와 궁합이 맞고, 다양한 플러그인이 오픈소스로 제공

### 특징

- 수집하는 Metric 데이터를 다차원의 시계열 데이터 형태로 저장
- 데이터 분석을 위한 자체 언어 PromQL 지원
- 시계열 데이터 저장에 적합한 TimeSeries DB 지원
- **데이터 수집하는 방식이 Pull 방식**
    - **모니터링 대상의 Agent 가 Server 로 Metric을 보내는 Push 방식이 아닌, Server 가 직접 정보를 가져가는 Pull 방식**
    - Push 방식을 위한 Push Gateway 도 지원
- 다양한 시각화 툴과의 연동 지원
- 다양한 방식의 Alarming 지원

### 구조

<!-- ![[https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab4388ab-243c-478f-9cb5-169b7a161067/Untitled.png) -->
![Untitled](/assets/img/mlops_pg_1.png)

[https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/)

- **Prometheus Server**
    - 시계열 데이터를 수집하고 저장
        - metrics 수집 주기를 설치 시 정할 수 있으며 default 는 15초
- **Service Discovery**
    - Monitoring 대상 리스트를 조회
    - 사용자는 쿠버네티스에 등록하고, Prometheus Server 는 쿠버네티스 API Server 에게 모니터링 대상을 물어보는 형태
- **Exporter**
    - Prometheus 가 metrics 을 수집해갈 수 있도록 정해진 HTTP Endpoint 를 제공하여 정해진 형태로 metrics 를 Export
    - Prometheus Server 가 이 Exporter 의 Endpoint 로 HTTP GET Request 를 보내 metrics 를 Pull 하여 저장한다.
    - 하지만, 이런 pull 방식은 수집 주기와 네트워크 단절 등의 이유로 모든 Metrics 데이터를 수집하는 것을 보장할 수 없기 때문에 Push 방식을 위한 Pushgateway 제공
- **Pushgateway**
    - 보통 Prometheus Server 의 pull 주기(period) 보다 짧은 lifecycle 을 지닌 작업의 metrics 수집 보장을 위함
- **AlertManager**
    - PromQL 을 사용해 특정 조건문을 만들고, 해당 조건문이 만족되면 정해진 방식으로 정해진 메시지를 보낼 수 있음
        - ex) service A 가 5분간 응답이 없으면, 관리자에게 slack DM 과 e-mail 을 보낸다.
- **Grafana**
    - Prometheus 와 항상 함께 연동되는 시각화 툴
    - Prometheus 자체 UI 도 있고, API 제공을 하기에 직접 시각화 대시보드를 구성할 수도 있음
- **PromQL**
    - Prometheus 가 저장한 데이터 중 원하는 정보만 가져오기 위한 Query Language 제공
    - Time Series Data 이며, Multi-Dimensional Data 이기 때문에 처음 보면 다소 복잡할 수 있으나 Prometheus 및 Grafana 를 잘 사용하기 위해서는 어느 정도 익혀두어야 함
    
    [Querying basics | Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)
    

### 단점

- Scalability, High Availability
    - Prometheus Sever 가 Single node 로 운영되어야 하기 때문에 발생하는 문제
- ⇒ Thanos 라는 오픈소스를 활용해 multi prometheus server 를 운영
    
    [Thanos](https://thanos.io/)
    

---

# 2. Grafana

[GitHub - grafana/grafana: The open and composable observability and data visualization platform. Visualize metrics, logs, and traces from multiple sources like Prometheus, Loki, Elasticsearch, InfluxDB, Postgres and many more.](https://github.com/grafana/grafana)

> The open and composable observability and data visualization platform.
- grafana
> 
- 2014 년 릴리즈된 프로젝트로 처음에는 InfluxDB, Prometheus 와 같은 TimeSeriesDB 전용 시각화 툴로 개발되었으나 이후 MySQL, PostgreSQL 과 같은 RDB 도 지원
- 현재는 Grafana Labs 회사에서 관리하고 있으며, 실습을 진행할 Open Source Project 인 **Grafana** 외에도 상용 서비스인 **Grafana Cloud**, **Grafana Enterprise** 제품 존재
    - 상용 서비스는 추가 기능을 제공하는 것뿐만 아니라 설치 및 운영 등의 기술 지원까지 포함
- playground 페이지도 제공하여 쉽고 간편하게 Grafana Dashboard 를 사용해볼 수 있음

[Grafana](http://play.grafana.org)

- 마찬가지로 쿠버네티스에 종속적이지는 않고 docker 로 쉽게 설치할 수는 있지만, 여러 Datasource 와의 연동성이 뛰어나고 특히 Prometheus 와의 연동이 뛰어나 함께 발전

### 다양한 외부 Plugins

[Grafana Plugins - extend and customize your Grafana](https://grafana.com/grafana/plugins/)

### 다양한 Grafana Dashboard

[Dashboards](https://grafana.com/grafana/dashboards/)

### Grafana Dashboard 모범 사례

- 수많은 Metric 중 모니터링해야할 대상을 정하고 어떤 방식으로 시각화할 것인지에 대한 정답은 없습니다. (Task 마다 달라지는 요구사항)
- 다만, Google 에서 제시한 전통 SW 모니터링을 위한 4 가지 황금 지표는 다음과 같음
    - Latency
        - 사용자가 요청 후 응답을 받기까지 걸리는 시간
    - Traffic
        - 시스템이 처리해야 하는 총 부하
    - Errors
        - 사용자의 요청 중 실패한 비율
    - Saturation
        - 시스템의 포화 상태
- ML 기반의 서비스를 모니터링할 때도 위 4가지 지표를 염두에 두고 대시보드를 구성하는 것을 권장
    - 다만 처음 시작할 때는 위의 다양한 오픈소스 대시보드 중 하나를 import 하는 것부터 시작