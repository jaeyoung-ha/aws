## 5. [Gremlin](https://www.gremlin.com) 시작하기

Gremlin은 Chaos Engineering 도구를 SaaS 서비스로 제공하는 회사이고 서비스 이름이기도 합니다.

![Gremlin](https://user-images.githubusercontent.com/6407492/78215124-c2a8bc80-74f1-11ea-8f6e-30d18dfd1107.png)

### 5.1. Amazon EKS에 Gremlin 설치하고 사용하기

* 아래 링크의 가이드에 따라, Amazon EKS에서 Gremlin을 사용해서 Chaos Engineering 실험을 할 수 있습니다.

* [https://www.gremlin.com/community/tutorials/how-to-install-and-use-gremlin-with-eks/](https://www.gremlin.com/community/tutorials/how-to-install-and-use-gremlin-with-eks/)

* 가이드에는 아래와 같은 사전 조건과 작업 항목이 정리되어 있습니다.
  
    * 사전 조건       
        * AWS 계정
        * AWS CLI
        * Gremlin 계정
        
    * 작업 항목
        * eksctl을 사용한 EKS 클러스터 생성
        * helm을 사용한 Gremlin 설치
        * Demo 애플리케이션 배포
        * Gremlin을 사용한 컨테이너 공격

* 관련 데모 영상입니다.

[![데모 영상](https://dykqnb76krm40.cloudfront.net/summit-2020/1.gremlin.png)](https://dykqnb76krm40.cloudfront.net/summit-2020/1.gremlin.mp4)

    `Gremlin 가이드로 실습이 가능하지만, 피드백에 따라 더 간단한 가이드를 작성할 예정입니다.`
  
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
