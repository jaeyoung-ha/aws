## 3. AWS App Mesh 시작하기

### 3.1. 정책(Policy) 추가

* Node IAM Role에 필요한 정책을 추가합니다.

   * App Mesh Controller를 위한 Policy 추가
    ```bash
    aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/AWSAppMeshFullAccess
    ```
     - IRSA(IAM Role for Service Account)을 사용하면, 더 정교한 권한 설정을 할 수 있습니다.
     [https://eksworkshop.com/advanced/320_servicemesh_with_appmesh/create_app_mesh_components/create_injector_controller/
](https://eksworkshop.com/advanced/320_servicemesh_with_appmesh/create_app_mesh_components/create_injector_controller/) 


   * X-Ray를 위한 Policy 추가  
    ```bash
    aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/AWSXrayFullAccess 
    ```  
   
   * Container Insights를 위한 Policy 추가   
    ```bash
    aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy 
    ```

   * Container Insights를 활성화 합니다.   
    (`Seoul(ap-northeast-2)` 리전이 아닌 경우, 리전코드를 변경합니다.)   
    
    curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/eksworkshop-eksctl/;s/{{region_name}}/ap-northeast-2/" | kubectl apply -f -


### 3.2. App Mesh Integration 설정

AWS App Mesh는 마이크로서비스 간의 네트워크 트래픽을 제어하고 가시성을 제공합니다.
Control Plane을 Managed Service로 제공하고, Data Plane에 배포된 Envoy sidecar proxy를 통해 사용자의 설정을 즉시 반영합니다.

<img width="988" alt="AWS-App-Mesh" src="https://user-images.githubusercontent.com/6407492/78212880-0f899480-74ec-11ea-94dc-38efc5c7fc1a.png">

<img width="756" alt="envoy" src="https://user-images.githubusercontent.com/6407492/78213167-c4bc4c80-74ec-11ea-95b9-9178be30fb6f.png">

Kubernetes에서 App Mesh를 사용하기 위해서 App Mesh Controller, App Mesh Sidecar Injector를 설치합니다.

* Helm CLI를 설치합니다.

    ```bash
    cd ~/environment 
    curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh 
    chmod +x get_helm.sh 
    ./get_helm.sh
    ```
    
* rbac.yaml    

    ```yaml
    cat <<EoF > ~/environment/rbac.yaml
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: tiller
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: tiller
        namespace: kube-system
    EoF
    ```
    
    ```bash
    kubectl apply -f ~/environment/rbac.yaml
    ```
    
    ```bash
    helm init --service-account tiller
    ```

* App Mesh를 위한 Namespace, CRDs를 생성합니다.

    ```bash
    helm repo add eks https://aws.github.io/eks-charts
    ```
    
    ```bash
    kubectl create ns appmesh-system
    ```
    
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/aws/eks-charts/master/stable/appmesh-controller/crds/crds.yaml
    ```

* App Mesh Controller를 설치합니다.

    ```bash
    helm upgrade -i appmesh-controller eks/appmesh-controller --namespace appmesh-system
    ```

* App Mesh Injector를 설치합니다.

    ```bash
    helm upgrade -i appmesh-inject eks/appmesh-inject \
    --namespace appmesh-system \
    --set mesh.create=true \
    --set mesh.name=appmesh-workshop
    ```
    

### 3.3. App Mesh 생성 확인

* appmesh-system 네임스페이스에 생성된 Kubernetes 오브젝트를 조회합니다.

    ```bash  
    kubectl -n appmesh-system get deploy,pods,service
    ```  
    
* 생성한 appmesh를 조회합니다.
    
    ```bash  
    aws appmesh list-meshes | jq '.meshes[].meshName'
    ``` 
    
    ```bash    
    kubectl -n bookinfo get meshes
    ``` 


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
