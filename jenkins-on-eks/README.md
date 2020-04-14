## [Amazon EKS](https://aws.amazon.com/ko/eks/)에서 [Jenkins](https://jenkins.io/)로 분산 빌드 환경 구성하기

본 실습에서는 EKS 클러스터 자원을 활용해서 빌드 작업을 실행하기 위해, Master/Slave 구조로 Jenkins를 구성합니다.     
Master는 빌드 작업 생성 및 스케줄링을 담당하고, Slave는 빌드 작업을 실행합니다.    
샘플 애플리케이션은 Java, Spring Boot 기반의 hello-spring-boot입니다.    
빌드/배포 파이프라인은 git clone -> mvn package -> docker build & push -> kubectl set image로 구성됩니다.    
추가적으로 [Cluster Autoscaler](https://eksworkshop.com/beginner/080_scaling/deploy_ca/)를 적용하면, 빌드 작업 요청에 따라서 클러스터 노드를 탄력적으로 구성할 수 있습니다.   

![Screen Shot 2020-04-15 at 7 23 47 AM](https://user-images.githubusercontent.com/6407492/79279917-15b74200-7eea-11ea-875b-777526c2484e.png)


### 사전 조건
본 실습을 진행하기 위해서 GitHub, DockerHub 계정과 Amazon EKS 클러스터 구성이 필요합니다.   
아래 링크의 가이드를 참조해서 Cloud9에서 Amazon EKS 클러스터를 생성합니다.

- [GitHub 계정 생성](https://github.com/join)
- [DockerHub 계정 생성](https://hub.docker.com/signup)
- [Amazon EKS 클러스터 생성](../summit-2020/100_amazon_eks.md)

### 1. [Jenkins 설치](100_install_jenkins.md)

Amazon EKS에 Jenkins를 설치합니다.  
EBS볼륨을 정적 또는 동적으로 바인딩해서 Jenkins를 설치합니다.    
EBS CSI Driver를 사용해서 Storage Class, Persistent Volume, Persistent Volume Claim을 생성합니다.    
Jenkins를 위한 Deployment, Service, Ingress를 배포합니다.   

### 2. [Jenkins 설정](200_configure_jenkins.md)

Amazon EKS에서 빌드/배포 작업을 실행할 수 있도록 Jenkins를 설정합니다.      
Jenkins 사용자이름/패스워드를 설정하고, [Kubernetes 플러그인](https://github.com/jenkinsci/kubernetes-plugin/blob/master/README.md)을 설치합니다.    
빌드 작업을 EKS 클러스터 노드로 분산하기 위해 Master/Slave(Agent) 구조의 [분산 빌드](https://wiki.jenkins.io/display/JENKINS/Distributed+builds) 환경을 구축합니다.     

### 3. [빌드 작업 생성 및 실행](300_build_job_creation.md)

빌드 작업을 생성하고 Amazon EKS에서 빌드 작업을 분산 실행합니다.   
빌드 작업을 구성하기 전에, GitHub Private Repository를 구성합니다.   
Jenkins에서 GitHub와 DockerHub Credential을 설정합니다.    
빌드 작업(git clone -> maven build -> docker build & push -> kubectl set image)을 생성하고 실행합니다.
