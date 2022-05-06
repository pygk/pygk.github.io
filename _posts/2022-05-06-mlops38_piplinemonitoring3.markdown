---
layout: single
title: 실습 3. Jenkins Monitoring
date: 2022-05-06 17:18:00 +0900
last_modified_at: 2022-05-06 17:18:00 +0900
category: mlops
tags: ["Pipeline Monitoring"]
published: false

---

- Jenkins Plugin 에서 Prometheus 추가
    - 환경설정 - Prometheus 확인
        - Collecting metrics period in seconds : 120 → 5 변경
- Jenkins-Promethues 확인
    - [localhost:8080/prometheus](http://localhost:8080/prometheus) 접속
    - prometheus.yml 내용 추가
        
        ```yaml
        - job_name: 'jenkins'
          metrics_path: /prometheus/
          static_configs:
            - targets:
              - localhost:8080
        ```
        
    - [http://localhost:9090/config](http://localhost:9090/config) 반영 확인
    - `docker restart prom-docker`
    - [http://localhost:9090/config](http://localhost:9090/config) 반영 확인
- Grafana Dashboard
    - [https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/) 접속
    - jenkins 검색
    - [Jenkins: Performance and Health Overview](https://grafana.com/grafana/dashboards/9964) 선택
    - ID 복사
    - Grafana - Create - Import - Import via [grafana.com](http://grafana.com/) 에 입력
    - 이전 강의 파이프라인 빌드 실행하여 변화 확인