# Containers & DevOps on AWS

많은 기업들이 서비스의 빠른 개발과 배포, 안정적인 서비스 운영을 위해서 마이크로서비스를 적용하고 있습니다.    
마이크로서비스를 통해 비즈니스 차별화에 더 집중하고, 새로운 서비스를 더 자주, 더 빠르게 딜리버리할 수 있습니다.   

그런데, 비즈니스가 점점 복잡해지는 것처럼, 마이크로서비스도 그 복잡도가 함께 증가하고 있습니다.    
수많은 마이크로서비스를 수월하게 관리하기 위해서 컨테이너, 컨테이너 오케스트레이션 툴이 필요하게 되었습니다.  

AWS는 자체적으로 개발한 컨테이너 오케스트레이션 툴, Amazon ECS(Elastic Container Service)와 Kubernetes(K8s)를 Managed Service로 제공하는 Amazon EKS(Elastic Kubernetes Service)를 제공합니다. AWS Fargate를 사용하면 클러스터 Node와 인프라 관리를 AWS에 맡기고 애플리케이션 개발에 더 집중할 수 있습니다. Fargate는 Amazon ECS와 Amazone EKS를 모두 지원합니다.

마이크로서비스 아키텍처는 많고 작은 애플리케이션들이 REST API와 같은 경량 네트워크를 통해 서로 통신하면서 동작합니다. 각 마이크로서비스 애플리케이션은 독립적으로 배포/실행/확장이 가능하지만, 복잡한 마이크로서비스 간의 네크워크 구조에 대한 파악할 수 없고 제어하지 못하면, 예측할 수 없는 장애가 생길 수 있습니다. 애플리케이션 레벨에서 네트워크 라우팅을 제어하고 가시성을 확보할 수 있는 Service Mesh 서비스로, AWS App Mesh를 제공합니다.

AWS의 다양한 컨테이너 서비스를 아래와 같이 제공합니다.  
![container1](https://user-images.githubusercontent.com/6407492/78220174-fb4d9380-74fb-11ea-9be0-a54de89ae79d.png)

워크로드의 요구사항이나 사용자의 기준에 따라, 사용할 인프라와 오케스트레이션 엔진을 다양하게 선택할 수 있습니다.
![container2](https://user-images.githubusercontent.com/6407492/78220177-fd175700-74fb-11ea-82e9-badfc8947283.png)

AWS는 마이크로서비스의 모니터링을 위해 CloudWatch Container Insights, FireLens, App Mesh 등의 서비스를 제공합니다.
![container3](https://user-images.githubusercontent.com/6407492/78220181-fe488400-74fb-11ea-83ba-0f511592a8d2.png)

Amazon ECS/EKS/Fargate와 App Mesh와 같은 AWS의 컨테이너 서비스를 사용하면, 컨테이너 기반의 마이크로서비스를 쉽고 안정적으로 관리할 수 있습니다. 그리고, AWS가 제공하는 글로벌 인프라, 리전, 다중 가용영역을 통해서 가용성, 안정성, 확장성 등의 클라우드의 이점을 그대로 상속받을 수 있습니다.

CodePipeline/CodeCommit/CodeBuild/CodeDeploy와 같은 CI/CD 서비스와 CloudFormation/CDK와 같은 IaC(Infrastructure as Code) 서비스를 사용하시면, 애플리케이션의 빌드/배포와 인프라 생성에 대한 자동화를 통해 더욱 빠르게 새로운 인프라와 서비스를 딜리버리할 수 있습니다.  
<img width="1229" alt="Screen Shot 2020-04-05 at 9 56 42 AM" src="https://user-images.githubusercontent.com/6407492/78464385-e2bfc200-7723-11ea-9b5e-4333a63b98d7.png">

CDK(Cloud Development Kit)을 사용하면, CloudFormation을 보다 쉽게 사용해서 인프라 생성을 할 수 있습니다.
![Screen Shot 2020-04-05 at 9 57 15 AM](https://user-images.githubusercontent.com/6407492/78464386-e5221c00-7723-11ea-9953-bdd5cda4ff13.png)

아마존의 Container와 DevOps 관련 컨텐츠를 공유하려고 합니다.  
방문해 주셔서 감사합니다.  

---
[Chaos Engineering을 활용한 마이크로서비스 안정성 개선하기](summit-2020)
