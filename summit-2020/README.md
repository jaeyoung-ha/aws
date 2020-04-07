## [Chaos Engineering](https://principlesofchaos.org/)을 활용한 [마이크로서비스](https://aws.amazon.com/ko/microservices/) 안정성 개선하기


## 실습의 목적 

  Chaos Engineering 도구를 활용해서 마이크로서비스에 실패를 주입하고, Amazon EKS와 App Mesh를 활용해서 마이크로서비스의 안정성과 복원력을 개선하는 방법을 알아봅니다.
  
  * [AWS](https://aws.amazon.com/ko) 클라우드에서 [Amazon EKS](https://aws.amazon.com/ko/eks/) 클러스터를 생성하고 샘플 마이크로서비스를 배포합니다.
  * 샘플 마이크로서비스는 [Book Info 애플리케이션](https://istio.io/docs/examples/bookinfo/)을 사용합니다.
  * [AWS App Mesh](https://aws.amazon.com/ko/app-mesh/)를 통해 마이크로서비스 간 네트워크 라우팅을 제어하고 [Container Insights](https://aws.amazon.com/ko/blogs/korea/operational-insights-for-containers-and-containerized-applications/)와 [X-Ray](https://aws.amazon.com/ko/xray/)를 활성화해서 가시성을 확보합니다.
  * [Gremlin](https://www.gremlin.com), [Litmus](https://litmuschaos.io)와 같은 Chaos Engineering 도구를 사용해서 마이크로서비스에 실패를 주입합니다. 
  * AWS App Mesh의 [Retry Policy](https://aws.amazon.com/ko/about-aws/whats-new/2019/09/aws-app-mesh-now-supports-retry-policies/)를 설정해서 복원력을 개선하는 방법을 알아봅니다.
 
본 실습 및 테스트는 `Seoul(ap-northeast-2)` region에서 진행하였습니다.


## 1. [Amazon EKS 시작하기](100_amazon_eks.md)

동일한 실습 환경을 사용하기 위해 Cloud9 workspace를 생성합니다.  
Kubernetes 관리를 위한 kubectl, aws cli, eksctl 등의 도구를 설치합니다.  
EKS 클러스터 구동에 필요한 환경정보를 설정하고, 클러스터를 생성합니다.  
클러스터 외부의 트래픽을 ALB를 통해서 받기 위해 ALB Ingress Controller를 설치합니다.


## 2. [Book Info 애플리케이션 배포하기](200_bookinfo_app.md)

샘플 마이크로서비스를 경험하기 위해 Book Info 애플리케이션을 배포합니다.  
Book Info 애플리케이션은 productpage, detail, review, rating 과 같은 마이크로서비스로 구성되어 있습니다.


## 3. [AWS App Mesh 시작하기](300_aws_app_mesh.md)

마이크로서비스 간의 네트워크 트래픽을 제어하고 가시성을 확보하기 위해 AWS App Mesh를 구성합니다.    
EKS에서 App Mesh를 사용하기 위해서 App Mesh Controller, App Mesh Injector를 설치합니다.    
Container Insights, X-Ray 활성화 등에 필요한 정책(Policy)을 Node Role에 합니다.    


## 4. [Book Info 애플리케이션에 App Mesh 리소스 배포하기](400_app_mesh_deploy.md)

Book Info 애플리케이션을 위한 App Mesh 리소스(Virtual Node, Virtual Service)를 배포합니다.    
App Mesh로 구성된 Book Info Application에 트래픽을 주입합니다.  
Container Insights, X-Ray 등을 통해 트래픽 정보를 모니터링 합니다.    


## 5. [Gremlin 시작하기](500_gremlin.md)

* 공식 홈페이지 : [https://www.gremlin.com](https://www.gremlin.com)

  * [Amazon EKS에서 Gremlin 설치 및 사용하기](https://www.gremlin.com/community/tutorials/how-to-install-and-use-gremlin-with-eks/)
    
  * [Gremlin 서비스 페이지에서 Attack 생성해서 공격하기](https://app.gremlin.com)


## 6. [Litmus 시작하기](600_litmus.md)

* 공식 홈페이지 : [https://litmuschaos.io](https://litmuschaos.io)

  * [Amazon EKS에서 Litmus 설치 및 사용하기](https://docs.litmuschaos.io/docs/getstarted/)
  
  * [Litmus ChaosEngine 카탈로그 살펴보기](https://hub.litmuschaos.io)


## 7. [Book Info 애플리케이션에 실패 주입하기](700_failure_injection.md)

Book Info 애플리케이션의 Ratings 서비스에 Litmus를 통해 Network Latency 실패를 주입합니다.    
App Mesh의 Retry Policy를 적용해서 복원력을 개선하는 방법을 알아봅니다.  


## 8. [리소스 삭제하기](800_cleanup.md)

실습을 통해 생성한 Book Info 애플리케이션, 서비스 메시, EKS 클러스터를 삭제합니다.

