## 8. 리소스 삭제하기

### 8.1. bookinfo namespace를 삭제합니다.

  ```bash
  kubectl delete namespace bookinfo
  ```

### 8.2. appmesh-workshop mesh를 삭제합니다.

  ```bash
  kubectl delete meshes appmesh-workshop
  ```

### 8.3. appmesh-inject, appmesh-controller helm charts를 삭제합니다.

  ```bash 
  helm delete appmesh-inject 
  helm delete appmesh-controller 
  ```

### 8.4. appmesh CRDs 삭제합니다.

  ```bash
  kubectl delete -f https://raw.githubusercontent.com/aws/eks-charts/master/stable/appmesh-controller/crds/crds.yaml
  ```

### 8.5. appmesh-system namespace 삭제합니다.

  ```bash
  kubectl delete namespace appmesh-system
  ```

### 8.6. eksworkshop-eksctl cluster 삭제합니다.

  ```bash
  eksctl delete cluster --name=eksworkshop-eksctl
  ```

### 8.7. Cloud9 workspace 삭제합니다.

  * [Cloud9 Environment](https://console.aws.amazon.com/cloud9/home)로 이동합니다.
  
  * eksworkshop을 선택하고 삭제(Delete) 버튼을 클릭합니다.


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
