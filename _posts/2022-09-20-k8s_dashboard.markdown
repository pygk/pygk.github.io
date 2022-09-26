---
layout: custom
title: k8s dashboard
date: 2022-09-20 15:34:00 +0900
last_modified_at: 2022-09-22 15:34:00 +0900
category: k8s
tags: ["k8s"]
published: False

---
> k8s dashboard

- pod 생성
    - 참고
        - [https://github.com/kubernetes/dashboard/tree/master/docs](https://github.com/kubernetes/dashboard/tree/master/docs)
        - [https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/)
        - [https://csupreme19.github.io/devops/kubernetes/2021/03/04/kubernetes-dashboard.html](https://csupreme19.github.io/devops/kubernetes/2021/03/04/kubernetes-dashboard.html)

    - 대시보드 UI 배포
    ```bash
    $ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
    $ mv recommended.yaml k8s-dashboard.yaml
    $ kubectl apply -f k8s-dashboard.yaml
    ```

    - 대시보드 접속 테스트
    ```bash
    $ kubectl proxy
    Starting to serve on 127.0.0.1:8001

    $ curl -v http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
    
    *   Trying 127.0.0.1...
    * TCP_NODELAY set
    * Connected to localhost (127.0.0.1) port 8001 (#0)
    > GET /api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/ HTTP/1.1
    > Host: localhost:8001
    > User-Agent: curl/7.58.0
    > Accept: */*
    >
    < HTTP/1.1 503 Service Unavailable
    < Audit-Id: aaf4c452-e504-4381-a093-9b007369e1b5
    < Cache-Control: no-cache, private
    < Content-Length: 217
    < Content-Type: application/json
    < Date: Thu, 22 Sep 2022 02:18:08 GMT
    <
    {
    "kind": "Status",
    "apiVersion": "v1",
    "metadata": {},
    "status": "Failure",
    "message": "no endpoints available for service \"https:kubernetes-dashboard:\"",
    "reason": "ServiceUnavailable",
    "code": 503
    * Connection #0 to host localhost left intact
    ```


    - 대시보드 UI 외부 접근 설정
    1. k8s-dashboard.yaml 수정
    ```
    ...
    spec:
        ports:
            - port: 80 # 수정  # http: 80, https: 443
                targetPort: {targetport} # 수정
        selector:
            k8s-app: kubernetes-dashboard
    ...
    containers:
        - name: kubernetes-dashboard
        image: kubernetesui/dashboard:v2.5.0
        imagePullPolicy: Always
        ports:
            - containerPort: {targetport} 
                protocol: TCP
        args:
            # - --auto-generate-certificates    # HTTPS 를 사용하지 않기 위해 주석 처리
            - --namespace=kubernetes-dashboard
            - --enable-skip-login=false # 추가
            - --enable-insecure-login=true # 추가, http 접속 허용
            - --insecure-bind-address=0.0.0.0 # 추가, 모든 ip 접속 허용
            - --token-ttl=3600 # 추가, 토큰 세션 유지 시간
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
        volumeMounts:
            - name: kubernetes-dashboard-certs
                mountPath: /certs
            # Create on-disk volume to store exec logs
            - mountPath: /tmp
                name: tmp-volume
        livenessProbe:
            httpGet:
                scheme: HTTPS
                path: /
                port: {targetport} # 수정
    ...
    ```

    2. dashboard 재배포
    ```bash
    $ kubectl apply -f k8s-dashboard.yaml
    namespace/kubernetes-dashboard unchanged
    serviceaccount/kubernetes-dashboard unchanged
    service/kubernetes-dashboard configured
    secret/kubernetes-dashboard-certs unchanged
    secret/kubernetes-dashboard-csrf unchanged
    secret/kubernetes-dashboard-key-holder unchanged
    configmap/kubernetes-dashboard-settings unchanged
    role.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
    clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
    rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
    clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
    deployment.apps/kubernetes-dashboard configured
    service/dashboard-metrics-scraper unchanged
    deployment.apps/dashboard-metrics-scraper unchanged
    ```

    3. Ingress yaml 파일 작성
    ```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
        name: kubernetes-dashboard-ingress
        annotations:
            nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
        rules:
        - http:
            paths:
            - path: /dashboard
                pathType: Prefix
                backend:
                    service:
                        name: kubernetes-dashboard
                        port:
                            number: 80
    ```

    3. Ingress 생성
    ```bash
    $ kubectl apply -f k8s-dashboard-ingress.yaml
    ingress.networking.k8s.io/kubernetes-dashboard-ingress created
    ```

    4. 계정생성
    ```
    # k8s-dashboard-user.yaml
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
        name: {admin}
        namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
        name: {admin}
    roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: cluster-admin
    subjects:
    - kind: ServiceAccount
        name: {admin}
        namespace: kube-system
    ```

    ```
    $ kubectl apply -f k8s-dashboard-user.yaml
    serviceaccount/{admin} created
    clusterrolebinding.rbac.authorization.k8s.io/{admin} created
    ```

    5. 계정 토큰 조회
    $ kubectl apply -f k8s-dashboard-user.yaml
    serviceaccount/{admin} created
    clusterrolebinding.rbac.authorization.k8s.io/{admin} created
    ```