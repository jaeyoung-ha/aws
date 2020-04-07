## 7. Book Info 애플리케이션에 실패 주입하기

Chaos Engineering 도구를 활용해서 Book Info 애플리케이션에 실패를 주입해 봅니다.  
Book Info 애플리케이션의 Ratings 서비스에 Litmus를 사용해서 네트워크 레이턴시 실패를 주입합니다.

<img width="869" alt="Book Info Istio" src="https://user-images.githubusercontent.com/6407492/78362020-52cf2a80-75f4-11ea-8d8b-a81e94f03f37.png">

### 7.1. Litmus를 통해 실패 주입하기

* Litmus Operator를 설치합니다. (6번 Litmus 실습에서 이미 설치했다면, 스킵해도 됩니다.)

    ```bash
    kubectl apply -f https://litmuschaos.github.io/pages/litmus-operator-v1.2.0.yaml
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 23 04 PM" src="https://user-images.githubusercontent.com/6407492/78345019-2d342800-75d8-11ea-89b6-3df6ffd356af.png"> 
     
* 설치된 Litmus 리소스를 확인합니다. (6번 Litmus 실습에서 이미 설치했다면, 스킵해도 됩니다.)
    
    ```bash
    kubectl get pods -n litmus
    ```
    <img width="1044" alt="Screen Shot 2020-04-03 at 6 23 38 PM" src="https://user-images.githubusercontent.com/6407492/78345260-75ebe100-75d8-11ea-9280-596f4b860336.png">  
     
    ```bash
    kubectl get crds | grep chaos
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 24 06 PM" src="https://user-images.githubusercontent.com/6407492/78345265-77b5a480-75d8-11ea-9505-4c9791af6131.png">  
     
    ```bash
    kubectl api-resources | grep chaos
    ```
    <img width="1044" alt="Screen Shot 2020-04-03 at 6 24 25 PM" src="https://user-images.githubusercontent.com/6407492/78345272-7a17fe80-75d8-11ea-8cca-48033975acf7.png">  
     
* Litmus Chaos Experiments를 설치합니다.
    
    ```bash
    kubectl apply -f https://hub.litmuschaos.io/api/chaos?file=charts/generic/experiments.yaml -n bookinfo
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 24 48 PM" src="https://user-images.githubusercontent.com/6407492/78345276-7be1c200-75d8-11ea-8af6-11bbc6e971bd.png">  
     
    ```bash
    kubectl get chaosexperiments -n bookinfo
    ```
    <img width="1044" alt="Screen Shot 2020-04-03 at 6 25 08 PM" src="https://user-images.githubusercontent.com/6407492/78345281-7dab8580-75d8-11ea-9319-e1cbb919faef.png">  
     
* Role, Role Binding, Service Account을 생성합니다.
    
    ```bash
    kubectl -n bookinfo apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-rbac-network-latency.yaml 
    ```
    <img width="1041" alt="Screen Shot 2020-04-03 at 6 28 28 PM" src="https://user-images.githubusercontent.com/6407492/78345929-48ebfe00-75d9-11ea-9afb-86e475a052e0.png">  
    
   * [bookinfo-rbac-network-latency.yaml](bookinfo/bookinfo-rbac-network-latency.yaml)  
     
   ```yaml
   ---
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: bookinfo-pod-network-latency-sa
     namespace: bookinfo
     labels:
       name: bookinfo-pod-network-latency-sa
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: Role
   metadata:
     name: bookinfo-pod-network-latency-sa
     namespace: bookinfo
     labels:
       name: bookinfo-pod-network-latency-sa
   rules:
   - apiGroups: ["","litmuschaos.io","batch"]
     resources: ["pods","jobs","pods/log","events","chaosengines","chaosexperiments","chaosresults"]
     verbs: ["create","list","get","patch","update","delete"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: RoleBinding
   metadata:
     name: bookinfo-pod-network-latency-sa
     namespace: bookinfo
     labels:
       name: bookinfo-pod-network-latency-sa
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: bookinfo-pod-network-latency-sa
   subjects:
   - kind: ServiceAccount
     name: bookinfo-pod-network-latency-sa
     namespace: bookinfo
   ```
      
* 공격 대상 애플리케이션(ratings-v1)에 아래와 같이 Annotation 설정을 합니다.
    
    ```bash
    kubectl annotate deploy/ratings-v1 litmuschaos.io/chaos="true" -n bookinfo
    ```
    <img width="1042" alt="Screen Shot 2020-04-03 at 6 28 58 PM" src="https://user-images.githubusercontent.com/6407492/78345935-4ab5c180-75d9-11ea-81b7-d7a0a96ae0c5.png">  
     
* 평점 정보를 가져오는 `ratings-v1 서비스에 네트워크 레이턴시를 주입`할 예정입니다.

  * 네트워크 지연이 발생하는 ratings-v1과 직접적으로 연결되는 `reviews-v3 Pod의 Port를 포워딩합니다.`
  * 트래픽 및 실패 주입을 모니터링 하기 위해, 아래와 같이 3개의 터미널을 각각 열어서 작업을 실행합니다.
  * `reviews-v3 Pod의 Port를 포워딩한 로컬 포트에 트래픽을 주입`하고 네트워크 지연으로 인해 문제가 발생하는지 살펴봅니다.
     
  * 터미널 1
    * Port 포워딩할 reviews-v3 Pod의 Name 정보를 조회합니다.
    
    ```bash
    kubectl get pod -n bookinfo
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 29 20 PM" src="https://user-images.githubusercontent.com/6407492/78345943-4d181b80-75d9-11ea-991a-d48d845d3b6c.png">  
         
    * review-v3의 Port를 로컬 Port로 포워딩합니다.
      
    ```bash
    kubectl -n bookinfo port-forward reviews-v3-686fdbf69b-vtghr 8080:9080
    ```
    <img width="1044" alt="Screen Shot 2020-04-03 at 6 29 41 PM" src="https://user-images.githubusercontent.com/6407492/78345947-4ee1df00-75d9-11ea-8974-37e1073c5b63.png">  
         
  * 터미널 2
    * Pod의 상태를 모니터링합니다.
      
    ```bash
    watch -n 1 kubectl get pods -n bookinfo
    ```
    <img width="607" alt="Screen Shot 2020-04-03 at 12 04 55 AM" src="https://user-images.githubusercontent.com/6407492/78269461-966a5b80-7544-11ea-820e-2f2b4b02b388.png">.  
         
   * 터미널 3
     * 터미널 1에서 포워딩 로컬 Port로 트래픽을 주입합니다.
      
    ```bash
    while true; do curl -s -o /dev/null -w "$(date +%H) $(date +%M) $(date +%S) %{http_code} %{time_total}" http://localhost:8080/reviews/0 ; sleep 0.5; echo ; done
    ```
    <img width="1045" alt="Screen Shot 2020-04-03 at 6 30 39 PM" src="https://user-images.githubusercontent.com/6407492/78345951-50aba280-75d9-11ea-8960-79f6e2b03517.png">  
         
* Pod Network Latency Chaos 실험을 실행합니다.

  * ratings Pod에 네트워크 레이턴시를 주입할 Litmus Chaos Pod을 배포합니다.
  * Litmus Chaos Pod의 `실행 시간은 30초`이고, ratings Pod에서 `3초의 네트워크 지연`이 발생합니다.
    
    ```bash
    kubectl -n bookinfo apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-chaos-network-latency.yaml
    ```
    <img width="1042" alt="Screen Shot 2020-04-03 at 6 35 28 PM" src="https://user-images.githubusercontent.com/6407492/78346490-0f67c280-75da-11ea-9e23-dc6ed4bd1da0.png">
    <img width="590" alt="Screen Shot 2020-04-03 at 12 07 03 AM" src="https://user-images.githubusercontent.com/6407492/78265364-4ccb4200-753f-11ea-9e42-9c9a66bc9c17.png">.  
     
  * [bookinfo-chaos-network-latency.yaml](bookinfo/bookinfo-chaos-network-latency.yaml)  
     
   ```yaml
   apiVersion: litmuschaos.io/v1alpha1
   kind: ChaosEngine
   metadata:
     name:  bookinfo-network-chaos
     namespace: bookinfo
   spec: 
     # It can be delete/retain
     jobCleanUpPolicy: 'delete'
     # It can be true/false
     annotationCheck: 'true'
     # It can be active/stop
     engineState: 'active'
     #ex. values: ns1:name=percona,ns2:run=nginx 
     auxiliaryAppInfo: ''
     monitoring: false
     appinfo: 
       appns: 'bookinfo'
       # FYI, To see app label, apply kubectl get pods --show-labels
       applabel: 'app=ratings'
       appkind: 'deployment'
     chaosServiceAccount: bookinfo-pod-network-latency-sa
     experiments:
       - name: pod-network-latency
         spec:
           components:
             env:
               #Container name where chaos has to be injected
               - name: TARGET_CONTAINER
                 value: 'ratings' 

               #Network interface inside target container
               - name: NETWORK_INTERFACE
                 value: 'eth0'     

               - name: LIB_IMAGE
                 value: 'gaiaadm/pumba:0.6.5'

               - name: NETWORK_LATENCY
                 value: '3000'

               - name: TOTAL_CHAOS_DURATION
                 value: '30' # in seconds
   ```
     
* Pod Network Latency Chaos 실험 결과를 확인합니다.

    ```bash
    kubectl describe chaosresult bookinfo-network-chaos-pod-network-latency -n bookinfo
    ```
    <img width="1046" alt="Screen Shot 2020-04-03 at 6 35 57 PM" src="https://user-images.githubusercontent.com/6407492/78346503-11ca1c80-75da-11ea-831e-0f61bf707cfa.png">  
     
* Book Info 애플리케이션에서 500 서버 에러가 발생하는 것을 확인합니다.
  
  * ratings Pod에 3초의 네트워크 지연이 발생하는데, 왜 500 에러가 발생할까요?
  * `reviews-v3에서 2.5초 이상의 지연이 발생하면, 타임아웃 처리`를 하도록 구현되었기 때문입니다.
  * reviews-v3의 백엔드, `ratings-v1에서의 3초의 지연`으로 `reviews-v3에서 타임아웃 에러`가 발생합니다.
    
  * 터미널 3
      
    ```bash
    while true; do curl -s -o /dev/null -w "$(date +%H) $(date +%M) $(date +%S) %{http_code} %{time_total}" http://localhost:8080/reviews/0 ; sleep 0.5; echo ; done
    ```
    <img width="176" alt="Screen Shot 2020-04-03 at 12 55 20 AM" src="https://user-images.githubusercontent.com/6407492/78270464-d251f080-7545-11ea-9f11-5abe37bc66ce.png">.   
    
* Pod Network Latency Chaos 실험을 삭제합니다.

    ```bash
    kubectl -n bookinfo delete -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-chaos-network-latency.yaml
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 36 18 PM" src="https://user-images.githubusercontent.com/6407492/78346510-1393e000-75da-11ea-913f-61d931618452.png"> 
     
### 7.2. Book Info 애플리케이션에 재시도 정책을 설정하고 재배포 합니다.

* [App Mesh의 재시도 정책(Retry Policy)](https://github.com/aws/aws-app-mesh-examples/tree/master/blogs/http-retry-policy)을 적용해 봅니다.   
    
    * 아래는 `ratings-v1으로 유입되는 트래픽를 제어`를 할 수 있는 Virtual Service입니다.  
    * `2초 간격으로 3번 재시도`하는 정책을 설정합니다.  
    
    ```yaml
   apiVersion: appmesh.k8s.aws/v1beta1
   kind: VirtualService
   metadata:
     name: ratings
     namespace: bookinfo
   spec:
     meshName: appmesh-workshop
     routes:
       - name: ratings-route
         http:
           match:
             prefix: /
           action:
             weightedTargets:
               - virtualNodeName: ratings-v1
                 weight: 1
           retryPolicy:
             maxRetries: 3
             perRetryTimeoutMillis: 2000
             httpRetryEvents:
               - server-error
    ``` 
    
* App Mesh Resources(Virtual Node, Virtual Service)를 재배포합니다. 
   
   * 이번 실습에서는 미리 생성한 yaml 파일을 실행합니다.

    ```bash
    kubectl -n bookinfo apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-controller-retry-policy.yml
    ```
    
### 7.3. 실행했던 Network Latency 실험을 반복합니다.

* 7.1 실습과 같은 방법으로 트래픽과 실패 주입을 모니터링합니다.
    
  * 터미널 2
      
   ```bash
   watch -n 1 kubectl get pods -n bookinfo
   ```
    <img width="607" alt="Screen Shot 2020-04-03 at 12 04 55 AM" src="https://user-images.githubusercontent.com/6407492/78269461-966a5b80-7544-11ea-820e-2f2b4b02b388.png">.  
       
  * 터미널 3
      
   ```bash
   while true; do curl -s -o /dev/null -w "$(date +%H) $(date +%M) $(date +%S) %{http_code} %{time_total}" http://localhost:8080/reviews/0 ; sleep 0.5; echo ; done
   ```
    <img width="1045" alt="Screen Shot 2020-04-03 at 6 30 39 PM" src="https://user-images.githubusercontent.com/6407492/78345951-50aba280-75d9-11ea-8960-79f6e2b03517.png">  
       
* Pod Network Latency Chaos 실험을 실행합니다.

  * ratings Pod에 네트워크 레이턴시를 주입할 Litmus Chaos Pod을 배포합니다.
  * Litmus Chaos Pod의 `실행 시간은 30초`이고, ratings Pod에서 `3초의 네트워크 지연`이 발생합니다.
  
    ```bash
    kubectl -n bookinfo apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-chaos-network-latency.yaml
    ```
    <img width="1042" alt="Screen Shot 2020-04-03 at 6 35 28 PM" src="https://user-images.githubusercontent.com/6407492/78346490-0f67c280-75da-11ea-9e23-dc6ed4bd1da0.png">  
    <img width="590" alt="Screen Shot 2020-04-03 at 12 07 03 AM" src="https://user-images.githubusercontent.com/6407492/78265364-4ccb4200-753f-11ea-9e42-9c9a66bc9c17.png">. 
    
* Pod Network Latency Chaos 실험 결과를 확인합니다.
    
    ```bash
    kubectl describe chaosresult bookinfo-network-chaos-pod-network-latency -n bookinfo
    ```
    <img width="1046" alt="Screen Shot 2020-04-03 at 6 35 57 PM" src="https://user-images.githubusercontent.com/6407492/78346503-11ca1c80-75da-11ea-831e-0f61bf707cfa.png">  
    
* Book Info 애플리케이션이 정상적으로 실행되는 것을 확인합니다.

  * 이번에는 `왜 reviews-v3에서 타임아웃 에러가 발생하지 않을까요?`
  * reviews-v3의 백엔드, ratings-v1에서의 3초 지연으로 reviews-v3에서 타임아웃 에러가 발생해도 `App Mesh 정책에 따라 재시도를 수행하여 정상적으로 요청을 처리합니다.`
    
  * 터미널 3
      
   ```bash
   while true; do curl -s -o /dev/null -w "$(date +%H) $(date +%M) $(date +%S) %{http_code} %{time_total}" http://localhost:8080/reviews/0 ; sleep 0.5; echo ; done
   ```
    <img width="157" alt="Screen Shot 2020-04-03 at 12 59 07 AM" src="https://user-images.githubusercontent.com/6407492/78270850-57d5a080-7546-11ea-9a8c-5a38364d186a.png">.  
    
* Pod Network Latency Chaos 실험을 삭제합니다.

    ```bash
    kubectl -n bookinfo delete -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-chaos-network-latency.yaml
    ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 36 18 PM" src="https://user-images.githubusercontent.com/6407492/78346510-1393e000-75da-11ea-913f-61d931618452.png"> 
     
     
### 7.4. Litmus를 삭제합니다.

* Litmus Chaos Experiments를 삭제합니다.

   ```bash
   kubectl delete -f https://hub.litmuschaos.io/api/chaos?file=charts/generic/experiments.yaml -n bookinfo
   ```
    <img width="1043" alt="Screen Shot 2020-04-03 at 6 40 13 PM" src="https://user-images.githubusercontent.com/6407492/78346849-987ef980-75da-11ea-9132-a78d347dc6fc.png">
  
* Litmus Operator를 삭제합니다.  
   
   ```bash
   kubectl delete -f https://litmuschaos.github.io/pages/litmus-operator-v1.2.0.yaml
   ```
    <img width="1044" alt="Screen Shot 2020-04-03 at 6 40 27 PM" src="https://user-images.githubusercontent.com/6407492/78346854-9b79ea00-75da-11ea-8d55-98cc1852cfa7.png">

* 관련 데모 영상입니다.

[![데모 영상](https://dykqnb76krm40.cloudfront.net/summit-2020/3.appmesh.png)](https://dykqnb76krm40.cloudfront.net/summit-2020/3.appmesh.mp4)   

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
