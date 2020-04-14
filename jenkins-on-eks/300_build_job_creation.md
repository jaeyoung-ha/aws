### 3.빌드 작업 생성

빌드 작업을 생성하고 Amazon EKS에서 빌드 작업을 분산 실행합니다.   
빌드 작업을 구성하기 전에, GitHub Private Repository를 구성합니다.   
Jenkins에서 GitHub와 DockerHub Credential을 설정합니다.    
빌드 작업(git clone -> maven build -> docker build & push -> kubectl set image)을 생성하고 실행합니다.

- GitHub **Private** Repository 생성
  - Repository 이름은 **jenkins-on-eks**로 입력합니다.   

![Screen Shot 2020-04-14 at 5 18 07 PM](https://user-images.githubusercontent.com/6407492/79202056-f978be00-7e73-11ea-99d5-94ad53f0af4e.png)  
 
  - Repository 생성이 완료되면, 앞에서 clone했던 파일들을 조금 전에 생성한 Repository로 commit & push합니다.
    - 아래 커맨드 중에서 **@username@과 @repository@는 여러분의 것으로 변경해야 합니다.**  
    
      ```
      cd ~/environment/jenkins-on-eks 
      git init
      git add .
      git commit -m "first commit"
      git remote set-url origin https://github.com/@username@/@repository@.git
      git push -u origin master
      ```
  
- **Docker Hub credentials** 생성  
  - 여러분의 Docker Hub 계정 Username과 Password를 입력합니다.  
  - ID는 **dockerhub**로 입력합니다.

    <img width="1187" alt="Screen Shot 2020-04-14 at 10 00 39 PM" src="https://user-images.githubusercontent.com/6407492/79227847-7e2b0280-7e9b-11ea-975f-dd5938830bf8.png"> 

- **GitHub credentials** 생성  
  - 여러분의 GitHub 계정 Username과 Password를 입력합니다.  
  - ID는 **github**로 입력합니다.  
  
    <img width="1189" alt="Screen Shot 2020-04-14 at 10 01 21 PM" src="https://user-images.githubusercontent.com/6407492/79227851-7ff4c600-7e9b-11ea-9ee2-9aee05a5899e.png">

- New Item 생성  
  
  - Item Name을 **test**로 입력하고, **Pipeline**을 선택합니다.  
  
<img width="1052" alt="Screen Shot 2020-04-13 at 7 59 13 PM" src="https://user-images.githubusercontent.com/6407492/79115619-451e5f80-7dc1-11ea-98fa-8bea85d94fd6.png">  
  
  - Pipeline script from SCM  
  
    - Pipeline 
      - SCM : **Git**
        - Repository (아래 URL에서 **@username@과 @repository@는 여러분의 것으로 변경해야 합니다.**)
          - Repository URL : **https://github.com/@username@/@repository@**
          - Credential : **조금 전에 생성한 GitHub Credential을 선택합니다.** 
      - Script Path : **jenkins/JenkinsFile**   
        (조금 전에 생성한 GitHub Repository에 jenkins 디렉토리에 JenkinsFile이 있는지 확인합니다.) 
    
<img width="888" alt="Screen Shot 2020-04-12 at 8 18 03 PM" src="https://user-images.githubusercontent.com/6407492/79067507-be9f4a80-7cfa-11ea-97f4-7464dd9f37c5.png">  

   - jenkins/JenkinsFile 
     - 파일 내용 중, **GitHub username(openzon), DockerHub username(openzon)을 여러분의 것으로 변경해야 합니다.**  
     
     
        ```bash
        node('jenkins-slave') {
          stage('maven build & docker build') {
              withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                  sh 'git clone https://$USERNAME:$PASSWORD@github.com/openzon/jenkins-on-eks.git'
                  //sh 'git clone git@github.com:openzon/jenkins-on-eks.git'
              }

              container('maven') {
                  stage('mvn install') {
                      dir('jenkins-on-eks/spring-boot') {
                          sh 'mvn clean package'
                      }
                  }
              }

              container('docker') {
                  stage('docker build') {
                      dir('jenkins-on-eks/spring-boot') {
                          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                          sh '''
                              docker build -t hello-spring-boot .
                              docker tag hello-spring-boot openzon/hello-spring-boot:${BUILD_NUMBER}
                              docker login -u=$USERNAME -p=$PASSWORD
                              docker push openzon/hello-spring-boot:${BUILD_NUMBER}
                          '''
                          }
                      }
                  }
              }
          }

          stage('deploy') {
             container('kubectl') {
                  stage('kubectl get pod') {
                      sh '''
                          kubectl -n default set image deployment/hello-spring-boot hello-spring-boot=openzon/hello-spring-boot:${BUILD_NUMBER} --record
                      '''
                  }
              }
          }
        }
        ```

- Sample Application 배포
  - 빌드 작업을 실행하기 전에, Amazon EKS에 Sample Application(hello-spring-boot)을 배포합니다.
  ```
  cd ~/environment/jenkins-on-eks
  kubectl apply -f jenkins/hello-spring-boot.yaml
  ```
  - Pod 배포 상태를 확인합니다.
  ```
  kubectl get pod
  ```
  
  - Ingress URL을 확인합니다.
  ```
  kubectl get ingress
  ```
  
  - Ingress URL에 접속하면, 아래와 같이 **Hello Spring Boot**이 출력되는 것을 확인할 수 있습니다.
  ```
  Hello Spring Boot
  ```
  
- 빌드 작업 실행
  - **Build Now**를 클릭해서 Build Job을 실행합니다.  
  
    <img width="1350" alt="Screen Shot 2020-04-14 at 10 26 38 PM" src="https://user-images.githubusercontent.com/6407492/79230151-22627880-7e9f-11ea-816a-7a75b1fe1aa7.png">

  - Pod 배포 상태와 Name을 확인합니다.
    ```
    kubectl get pod
    ```
  
  - Pod Name으로 상세정보를 조회합니다.
    - Container Image Tag가 Jenkins의 Build Number와 같은지 확인합니다.
    - 아래 커맨드의 **Pod Name을 여러분의 실제 Pod Name으로 변경해야 합니다.**  
    
      ```
      kubectl describe pod hello-spring-boot-f98fdddb-xtnm4
      ```
      
      <img width="795" alt="Screen Shot 2020-04-14 at 10 27 26 PM" src="https://user-images.githubusercontent.com/6407492/79230155-242c3c00-7e9f-11ea-8be0-4af42bcabef5.png">  
      
  
- 리소스 삭제

  - Jenkins 네임스페이스 삭제
  ```
  kubectl delete namespace jenkins
  ```
  
  - EKS 클러스터 삭제
  ```
  eksctl delete cluster --name=eksworkshop-eksctl
  ```
    
감사합니다.

---
### Table of Contents
[Jenkins on Amazon EKS](README.md)
1. [Jenkins 설치](100_install_jenkins.md)
2. [Jenkins 설정](200_configure_jenkins.md)
3. [빌드 작업 생성 및 실행](300_build_job_creation.md)