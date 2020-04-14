## 4. Book Info 에 App Mesh 리소스 배포하기

![App-Mesh-Object](https://user-images.githubusercontent.com/6407492/78213443-9a1ec380-74ed-11ea-832b-0aef5b7a4408.png)

### 4.1. App Mesh 리소스를 배포합니다.

* App Mesh 리소스(Virtual Node, Virtual Service)를 배포합니다.

    ```bash
    kubectl -n bookinfo apply -f https://raw.githubusercontent.com/openzon/aws/master/summit-2020/bookinfo/bookinfo-controller.yml
    ```

* [bookinfo-controller.yml](bookinfo/bookinfo-controller.yml)

* Mesh - appmehs-workshop

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: Mesh
    metadata:
      name: appmesh-workshop

    ```

* VirtualService - productpage

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualService
    metadata:
      name: productpage
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      routes:
        - name: productpage-route
          http:
            match:
              prefix: /
            action:
              weightedTargets:
                - virtualNodeName: productpage-v1
                  weight: 1

    ```

* VirtualService - reviews

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualService
    metadata:
      name: reviews
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      routes:
        - name: reviews-route
          http:
            match:
              prefix: /
            action:
              weightedTargets:
                #- virtualNodeName: reviews-v1
                #  weight: 1
                - virtualNodeName: reviews-v2
                  weight: 1
                - virtualNodeName: reviews-v3
                  weight: 1 

    ```

* VirtualService - ratings

    ```yaml
    ---
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
                  
    ```

* VirtualService - details

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualService
    metadata:
      name: details
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      routes:
        - name: details-route
          http:
            match:
              prefix: /
            action:
              weightedTargets:
                - virtualNodeName: details-v1
                  weight: 1
    ```

* VirtualNode - productpage-v1

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: productpage-v1
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      backends:
        - virtualService:
            virtualServiceName: details
        - virtualService:
            virtualServiceName: reviews
    #    - virtualService:
    #        virtualServiceName: ratings
      serviceDiscovery:
        dns:
          hostName: productpage.bookinfo.svc.cluster.local
    ```

* VirtualNode - reviews

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: reviews
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      serviceDiscovery:
        dns:
          hostName: reviews.bookinfo.svc.cluster.local
    ```

* VirtualNode - reviews-v1

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: reviews-v1
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      serviceDiscovery:
        dns:
          hostName: reviews-v1.bookinfo.svc.cluster.local
    ```

* VirtualNode - reviews-v2

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: reviews-v2
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      backends:
        - virtualService:
            virtualServiceName: ratings
      serviceDiscovery:
        dns:
          hostName: reviews-v2.bookinfo.svc.cluster.local
    ```

* VirtualNode - reviews-v3

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: reviews-v3
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      backends:
        - virtualService:
            virtualServiceName: ratings
      serviceDiscovery:
        dns:
          hostName: reviews-v3.bookinfo.svc.cluster.local
    ```

* VirtualNode - ratings-v1

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: ratings-v1
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      serviceDiscovery:
        dns:
          hostName: ratings.bookinfo.svc.cluster.local
    ```

* VirtualNode - details-v1

    ```yaml
    ---
    apiVersion: appmesh.k8s.aws/v1beta1
    kind: VirtualNode
    metadata:
      name: details-v1
      namespace: bookinfo
    spec:
      meshName: appmesh-workshop
      listeners:
        - portMapping:
            port: 9080
            protocol: http
      serviceDiscovery:
        dns:
          hostName: details.bookinfo.svc.cluster.local
    ```

* Virtual Node, Virtual Service의 배포상태를 확인합니다.
    
    ```bash
    kubectl get all -n bookinfo
    ```
    <img width="813" alt="Screen Shot 2020-04-03 at 1 49 56 PM" src="https://user-images.githubusercontent.com/6407492/78342992-26f07c80-75d5-11ea-95a0-a149e617c52e.png">  
    <img width="650" alt="Screen Shot 2020-04-03 at 1 40 00 PM" src="https://user-images.githubusercontent.com/6407492/78324784-a586f300-75b0-11ea-8d18-9296d5ce2eef.png">  

* Pod의 Container 개수를 확인합니다. (1/1)
    
    ```bash
    kubectl get pod -n bookinfo
    ```
    <img width="601" alt="Screen Shot 2020-04-03 at 1 09 39 PM" src="https://user-images.githubusercontent.com/6407492/78323311-65257600-75ac-11ea-89ca-7da4e24acccc.png">  
 
* Envoy sidecar proxy 주입을 위한 설정을 합니다.
    
    ```bash
    kubectl label namespace bookinfo appmesh.k8s.aws/sidecarInjectorWebhook=enabled
    ```             

* Envoy sidecar proxy 설정 반영을 위해서 Pod를 재시작합니다.
    
    ```bash    
    kubectl delete pod --all -n bookinfo
    ```    

* Envoy sidecar proxy 배포를 확인합니다. (2/2)

    ```bash    
    kubectl get pod -n bookinfo
    ```   
    <img width="586" alt="Screen Shot 2020-04-03 at 1 10 13 PM" src="https://user-images.githubusercontent.com/6407492/78323355-871ef880-75ac-11ea-9e11-d20d88c92c07.png">  

* X-Ray를 활성화 합니다.
    
    ```bash
    helm upgrade -i appmesh-inject eks/appmesh-inject \
    --namespace appmesh-system \
    --set mesh.create=false \
    --set mesh.name=appmesh-workshop \
    --set tracing.enabled=true \
    --set tracing.provider=x-ray 
    ```
        
* X-Ray 설정 반영을 위해서 Pod를 재시작합니다.
    
    ```bash    
    kubectl delete pod --all -n bookinfo
    ```    

* X-Ray daemon 배포를 확인합니다. (3/3)

    ```bash    
    kubectl get pod -n bookinfo
    ```   
    <img width="607" alt="Screen Shot 2020-04-03 at 1 10 35 PM" src="https://user-images.githubusercontent.com/6407492/78323359-88e8bc00-75ac-11ea-97a3-8e77a2fa8029.png"> 
    
### 4.2. Book Info 애플리케이션에 트래픽을 보냅니다.

* Ingress URL을 확인해서 아래 @IngressURL@ 부분을 변경합니다.
    
    ```bash 
    while true; do curl -o /dev/null -s "https://@IngressURL@/productpage"; done
    ```  

### 4.3. Container Insights를 통해 시스템 사용정보를 모니터링 합니다. 

* **Seoul(ap-northeast-2)** 리전이 아닌 경우, 리전코드를 변경합니다.
* [https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#cw:dashboard=Container](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#cw:dashboard=Container)
![Container-Insights](https://user-images.githubusercontent.com/6407492/78211329-9ee07900-74e7-11ea-9c88-6f35ea333d30.png)

### 4.4. X-Ray에서 마이크로서비스 간 호출 관계를 확인합니다. 

* **Seoul(ap-northeast-2)** 리전이 아닌 경우, 리전코드를 변경합니다.
* [https://ap-northeast-2.console.aws.amazon.com/xray/home?region=ap-northeast-2#/service-map](https://ap-northeast-2.console.aws.amazon.com/xray/home?region=ap-northeast-2#/service-map)
![X-Ray](https://user-images.githubusercontent.com/6407492/78211382-bd467480-74e7-11ea-9074-904fbe0fe0b5.png)


---
`※ 실습 가이드의 원본은` [github](https://github.com/openzon/aws/tree/master/summit-2020)`에 있습니다. 링크도 같이 참조해 주세요`   
  `github markdown -> github page html 전환 중에, 글자가 누락되거나 스타일 구성에 오류가 있을 수 있으니 양해 부탁드립니다.`
  
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
