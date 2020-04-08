## 1. Amazon EKS 시작하기

본 실습에서는 [eksworkshop.com](https://eksworkshop.com)의 가이드를 최대한 활용하고 있습니다.    
eksworkshop에 이미 Amazon EKS의 기능에 대한 다양한 실습 가이드가 제공되고 있고, 현재도 계속 최신 정보가 업데이트되고 있습니다.   
가이드 중 필요한 부분을 링크로 추가하였고, 수정이 필요한 가이드는 본 실습 가이드에 본문에 내용을 추가하였습니다.   
순서대로 따라 하시면 Amazon EKS 클러스터 생성, 샘플 마이크로서비스 배포, App Mesh 설정과 사용 등을 직접 경험할 수 있습니다.

Amazon EKS의 클러스터를 생성하는 표준 툴은 eksctl 입니다.  
eksctl를 사용해서 클러스터를 생성합니다.

### eksctl 관련 참조 문서

* eksctl 공식 홈페이지
  * [https://eksctl.io/](https://eksctl.io/)
  
* eksctl github
  * [https://github.com/weaveworks/eksctl](https://github.com/weaveworks/eksctl)
  
* AWS 개발자가이드 중 eksctl 시작하기
  * [https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html)

### 1.1. Cloud9 workspace 생성

* Cloud9은 AWS가 제공하는 통합 개발 환경(IDE) 입니다.

<img width="1437" alt="Screen Shot 2020-04-03 at 10 14 11 AM" src="https://user-images.githubusercontent.com/6407492/78314113-eb818e00-7593-11ea-85b3-64535aaa0ea0.png">
  
* 아래 링크로 이동해서 Cloud9 workspace를 생성할 준비를 합니다.
  * 이번 실습에서는 `Seoul(ap-northeast-2)` 리전에서 Cloud9 workspace를 생성합니다.
  * [https://ap-northeast-2.console.aws.amazon.com/cloud9/home?region=ap-northeast-2](https://ap-northeast-2.console.aws.amazon.com/cloud9/home?region=ap-northeast-2)
  
* Cloud9 workspace를 생성합니다.
  * [https://eksworkshop.com/020_prerequisites/workspace/](https://eksworkshop.com/020_prerequisites/workspace/)
  
* Kubernetes Tools를 설치합니다.
  * [https://eksworkshop.com/020_prerequisites/k8stools/](https://eksworkshop.com/020_prerequisites/k8stools/)
  
* Cloud9 workspace를 위한 IAM Role을 생성합니다.
  * [https://eksworkshop.com/020_prerequisites/iamrole/](https://eksworkshop.com/020_prerequisites/iamrole/)
  
* 생성한 IAM Role을 Cloud9 workspace EC2 인스턴스에 지정합니다.
  * [https://eksworkshop.com/020_prerequisites/ec2instance/](https://eksworkshop.com/020_prerequisites/ec2instance/)
  
* Cloud9 workspace를 위한 IAM 설정을 변경합니다.
  * [https://eksworkshop.com/020_prerequisites/workspaceiam/](https://eksworkshop.com/020_prerequisites/workspaceiam/)
  
  
### 1.2. EKS 클러스터 생성  

* 클러스터 생성 툴인 eksctl를 설치합니다.
  * [https://eksworkshop.com/030_eksctl/prerequisites/](https://eksworkshop.com/030_eksctl/prerequisites/)
  
* eksctl를 사용해서 EKS 클러스터를 생성합니다. (약 15분 소요)
  * 클러스터 이름 : `eksworkshop-eksctl`  

  ```bash
  # 클러스터 이름을 변경하면, 이후 작업에 영향이 있을 수 있습니다. 
  eksctl create cluster --name=eksworkshop-eksctl --nodes=3 --managed --alb-ingress-access --region=${AWS_REGION}
  ```
<img width="1085" alt="EKS-Cluster" src="https://user-images.githubusercontent.com/6407492/78212153-e7009b00-74e9-11ea-9812-3a9a077ae412.png">

* 정상적으로 Node가 생성되었는지 확인합니다.

  ```bash
  kubectl get nodes # if we see our 3 nodes, we know we have authenticated correctly
  ```
  
* Node IAM Role Name을 Export해서 `.bash_profile`에 추가합니다. 
  * Node IAM Role에 정책(Policy)을 추가할 때, 아래 설정이 사용됩니다.  
  
  ```bash
  STACK_NAME=$(eksctl get nodegroup --cluster eksworkshop-eksctl -o json | jq -r '.[].StackName')
  ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq -r '.StackResources[] |  select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
  echo "export ROLE_NAME=${ROLE_NAME}" | tee -a ~/.bash_profile
  ```

### 1.3. ALB Ingress Controller 설치

* ALB Ingress Controller를 통해서 Amazon ALB(Applicaton Load Balancer)를 Ingress 용도로 사용할 수 있습니다.
<img width="857" alt="ALB-Ingress-Controller" src="https://user-images.githubusercontent.com/6407492/78212032-9a1cc480-74e9-11ea-8874-f894174340a9.png">

* ALB Ingress Controller를 설치하고, Sample Application(2048-game)을 배포합니다.

  * [https://eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/](https://eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/)
  

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
