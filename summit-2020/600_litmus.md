## 6. [Litmus](https://litmuschaos.io) 시작하기

Litmus는 Kubernetes 환경에서 Chaos Engineering을 연습하고 실행할 수 있는 도구를 제공합니다. 

<img width="1308" alt="Litmus" src="https://user-images.githubusercontent.com/6407492/78216012-de14c700-74f3-11ea-82c7-1304945691ae.png">

### 6.1. Amazon EKS에 Litmus를 설치하고 사용하기

* 아래 링크의 가이드에 따라, Amazon EKS에서 Litmus를 사용해서 Chaos Engineering 실험을 할 수 있습니다.

* [https://docs.litmuschaos.io/docs/getstarted/](https://docs.litmuschaos.io/docs/getstarted/)

  * 사전 조건       
    * Kubernetes 1.11 이상
    * [Amazon EKS 클러스터 생성](1.%20Amazon%20EKS.md)
    
  * 가이드에는 아래와 같은 작업 항목이 정리되어 있습니다.
    * Litmus 설치
    * Chaos Experiments 설치
    * Service Account 설정
    * 애플리케이션 Annotation 설정
    * ChaosEngine 준비 및 작성
    * Chaos 실험 실행
    * 실험 결과 확인

* 관련 데모 영상입니다.

[![데모 영상](https://dykqnb76krm40.cloudfront.net/summit-2020/2.litmus.png)](https://dykqnb76krm40.cloudfront.net/summit-2020/2.litmus.mp4)

  `Litmus 가이드로 실습이 가능하지만, 피드백에 따라 더 간단한 가이드를 작성할 예정입니다.`


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
