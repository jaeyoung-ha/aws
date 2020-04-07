## 2. Book Info 애플리케이션 배포하기

![Bookinfo-Application](https://user-images.githubusercontent.com/6407492/78212695-735f8d80-74eb-11ea-810a-e66df008af31.png)

### 2.1. Namespace를 생성합니다.

* Book Info 애플리케이션을 위한 Namespace를 생성합니다.
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-ns-only.yml 
    ```
    
* [bookinfo-ns-only.yml](bookinfo/bookinfo-ns-only.yml)

    ```yaml
    ---
    apiVersion: v1
    kind: Namespace
    metadata:
      name: bookinfo
    ```

### 2.2. Deployment, Service, Ingress를 생성합니다.

* Book Info 애플리케이션을 배포합니다.

    ```bash   
    kubectl apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-app.yml -n bookinfo 
    ```  
     
* [bookinfo-app.yml](bookinfo/bookinfo-app.yml)


    * Service - details

    ```yaml
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: details
      namespace: bookinfo
      labels:
        app: details
        service: details
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: details
    ```

    * Deployment - details-v1

    ```yaml
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: details-v1
      namespace: bookinfo
      labels:
        app: details
        version: v1
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: details
            version: v1
        spec:
          containers:
          - name: details
            image: istio/examples-bookinfo-details-v1:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```
    

    * Service - ratings

    ```yaml
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: ratings
      namespace: bookinfo
      labels:
        app: ratings
        service: ratings
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: ratings    
    ```

    * Deployment - ratings-v1

    ```yaml
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: ratings-v1
      namespace: bookinfo
      labels:
        app: ratings
        version: v1
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: ratings
            version: v1
        spec:
          containers:
          - name: ratings
            image: istio/examples-bookinfo-ratings-v1:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```
    
    * Service - reviews

    ```yaml    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: reviews
      namespace: bookinfo
      labels:
        app: reviews
        service: reviews
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: reviews
    ```

    * Service - reviews-v1

    ```yaml   
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: reviews-v1
      namespace: bookinfo
      labels:
        app: reviews
        service: reviews
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: reviews
        version: v1   
    ```

    * Service - reviews-v2

    ```yaml 
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: reviews-v2
      namespace: bookinfo
      labels:
        app: reviews
        service: reviews
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: reviews
        version: v2
    ```

    * Service - reviews-v3

    ```yaml   
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: reviews-v3
      namespace: bookinfo
      labels:
        app: reviews
        service: reviews
    spec:
      ports:
      - port: 9080
        name: http
      selector:
        app: reviews
        version: v3
    ```

    * Deployment - reviews-v1

    ```yaml   
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: reviews-v1
      namespace: bookinfo
      labels:
        app: reviews
        version: v1
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: reviews
            version: v1
        spec:
          containers:
          - name: reviews
            image: istio/examples-bookinfo-reviews-v1:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```

    * Deployment - reviews-v2

    ```yaml 
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: reviews-v2
      namespace: bookinfo
      labels:
        app: reviews
        version: v2
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: reviews
            version: v2
        spec:
          containers:
          - name: reviews
            image: istio/examples-bookinfo-reviews-v2:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```

    * Deployment - reviews-v3

    ```yaml 
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: reviews-v3
      namespace: bookinfo
      labels:
        app: reviews
        version: v3
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: reviews
            version: v3
        spec:
          containers:
          - name: reviews
            image: istio/examples-bookinfo-reviews-v3:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```
    
    * Ingress - productpage

    ```yaml 
    ---
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: productpage
      namespace: bookinfo
      annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/scheme: internet-facing
      labels:
        app: productpage
    spec:
      rules:
        - http:
            paths:
              - path: /*
                backend:
                  serviceName: productpage
                  servicePort: 80

    ```

    * Service - productpage

    ```yaml 
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: productpage
      namespace: bookinfo
      labels:
        app: productpage
        service: productpage
    spec:
      ports:
      - port: 80
        targetPort: 9080
        name: tcp
      type: NodePort  
      selector:
        app: productpage

    ```

    * Deployment - productpage-v1

    ```yaml
    ---
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: productpage-v1
      namespace: bookinfo
      labels:
        app: productpage
        version: v1
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: productpage
            version: v1
        spec:
          containers:
          - name: productpage
            image: istio/examples-bookinfo-productpage-v1:1.15.0
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 9080
    ```
    
### 2.3. 생성한 Ingress 정보를 확인합니다.

* Ingress URL을 확인하고 브라우저를 통해 접속합니다. (ALB 활성화에 1~2분 정도가 소요됩니다.)

    ```bash 
    kubectl get ingress -n bookinfo 
    ```

![Bookinfo-Screen](https://user-images.githubusercontent.com/6407492/78212727-8e320200-74eb-11ea-9346-d99258f6a9ea.png)



---
### Table of Contents
[Chaos Engineering을 활용한 마이크로서비스 안정성 개선하기](README.md)
1. [Amazon EKS 시작하기](100_amazon_eks.md)
2. [Book Info 애플리케이션에 배포하기](200_bookinfo_app.md)
3. [AWS App Mesh 시작하기](300_aws_app_mesh.md)
4. [Book Info 애플리케이션에 App Mesh 리소스 배포하기](400_app_mesh_deploy.md)
5. [Gremlin 시작하기](500_gremlin.md)
6. [Litmus 시작하기](600_litmus.md)
7. [Book Info 애플리케이션에 실패 주입하기](700_failure_injection.md)
8. [리소스 삭제하기](800_cleanup.md)
