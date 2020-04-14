### 2. Jenkins 설정

Amazon EKS에서 빌드/배포 작업을 실행할 수 있도록 Jenkins를 설정합니다.      
Jenkins 사용자이름/패스워드를 설정하고, [Kubernetes 플러그인](https://github.com/jenkinsci/kubernetes-plugin/blob/master/README.md)을 설치합니다.    
빌드 작업을 EKS 클러스터 노드로 분산하기 위해 Master/Slave(Agent) 구조의 [분산 빌드](https://wiki.jenkins.io/display/JENKINS/Distributed+builds) 환경을 구축합니다.  

- Admin 패스워드 확인
  - Jenkins Pod Name을 조회합니다.
  ```
  kubectl -n jenkins get pod
  ```
  - Jenkins Pod Name으로 Pod Log를 출력해서 Admin 패스워드를 확인합니다.  
    (*Pod Name을 여러분의 것으로 변경해야 합니다.*)
  ```
  kubectl logs -n jenkins jenkins-59cdf95994-q4mkb
  ```
  ![Screen Shot 2020-04-13 at 3 16 04 PM](https://user-images.githubusercontent.com/6407492/79097051-bcd89400-7d99-11ea-992a-46557e5e1375.png)

- Jenkins 접속
  - Ingress URL 확인해서 Jenkins에 접속합니다.
  ```
  kubectl get ingress -n jenkins
  ```
  - Admin 패스워드를 입력합니다.  
  
  ![Screen Shot 2020-04-13 at 3 20 23 PM](https://user-images.githubusercontent.com/6407492/79097337-7e8fa480-7d9a-11ea-98a7-6efff6a5e9f2.png)  
  
  - Suggested plugins을 설치합니다.  
  
  ![Screen Shot 2020-04-13 at 3 20 38 PM](https://user-images.githubusercontent.com/6407492/79097366-936c3800-7d9a-11ea-9110-6596b587b0c3.png)
  
  - Plugins 설치가 완료될 때까지 기다립니다.  
  
  ![Screen Shot 2020-04-13 at 3 20 55 PM](https://user-images.githubusercontent.com/6407492/79097343-82232b80-7d9a-11ea-8de6-18c9a5f272f0.png)
  
- Username/Password 설정
  - Username : **jenkins**
  - Password : **jenkins** 
  - 위와 같이 Username과 Password를 입력합니다.  
  
  ![Screen Shot 2020-04-13 at 3 26 38 PM](https://user-images.githubusercontent.com/6407492/79097644-358c2000-7d9b-11ea-9cda-8351538860e5.png)
  
  - Jenkins URL을 확인합니다.  
  
  <img width="913" alt="Screen Shot 2020-04-14 at 9 40 09 PM" src="https://user-images.githubusercontent.com/6407492/79225983-92213500-7e98-11ea-8085-03949e1a7d80.png">  
  
  - Jenkins 설치가 완료되었습니다.  
  
  ![Screen Shot 2020-04-13 at 3 29 12 PM](https://user-images.githubusercontent.com/6407492/79097799-8f8ce580-7d9b-11ea-9dfc-b80c4923c9b7.png)
  
- Kubernetes Plugin 설치  

  - Kubernetes Plugin을 설치합니다.  
  
  ![Screen Shot 2020-04-13 at 3 49 32 PM](https://user-images.githubusercontent.com/6407492/79099123-66218900-7d9e-11ea-8699-9dcd50eb9fc4.png)
  
  - 설치가 완료될 때까지 기다립니다. 
  
  ![Screen Shot 2020-04-13 at 3 51 13 PM](https://user-images.githubusercontent.com/6407492/79125748-9f2b1f00-7dd9-11ea-90ab-503b612374e2.png)
  
  - Restart 체크박스를 클릭해서 Jenkins를 재기동합니다.  
  
  ![Screen Shot 2020-04-13 at 3 52 17 PM](https://user-images.githubusercontent.com/6407492/79099289-c7495c80-7d9e-11ea-84c6-80165dbcd706.png)

  - Jenkins에 다시 로그인합니다.  
  
  <img width="907" alt="Screen Shot 2020-04-14 at 9 45 29 PM" src="https://user-images.githubusercontent.com/6407492/79226398-4a4edd80-7e99-11ea-9103-05d8f95f1287.png">
  
  
- Manage Nodes and Clouds 설정  

  - 좌측 메뉴에서 **Manage Jenkins**를 클릭해서 화면을 이동합니다.  
  - **System Configuration** 영역에서 **Manage Node and Clouds**를 클릭합니다.   
  
     <img width="1470" alt="Screen Shot 2020-04-12 at 8 00 46 PM" src="https://user-images.githubusercontent.com/6407492/79067133-50f21f00-7cf8-11ea-89e1-ebc09ffdbf23.png">    
  
  - **Manage Nodes and Clouds** 화면에서 좌측 메뉴 **Configure Clouds**를 클릭해서 화면을 이동합니다.   
  
     <img width="1472" alt="Screen Shot 2020-04-12 at 8 02 10 PM" src="https://user-images.githubusercontent.com/6407492/79067174-83038100-7cf8-11ea-83db-10a2d7fa838c.png">  
    
  - **Configure Clouds**화면에서 **Add a new cloud**를 클릭해서 Kubernetes 정보를 아래와 같이 입력합니다.
    - Kubernetes
      - Name : **kubernetes**
      - Kubernetes URL : **https://kubernetes.default:443**
      - Kubernetes Namespace : **jenkins**
      - Jenkins URL : **http://jenkins**
      - Jenkins tunnel : **jenkins:50000**
        
        <img width="1470" alt="Screen Shot 2020-04-12 at 8 04 24 PM" src="https://user-images.githubusercontent.com/6407492/79067265-0a50f480-7cf9-11ea-8472-2ca577fe088b.png"> 
    
    - `Add Pod Label`을 클릭해서 Pod 정보를 아래와 같이 입력합니다.      
      - Pod Labels
        - Key : **jenkins**
        - Value : **slave**   
      - Pod Templates
        - Name : **jenkins-slave** 
        - Namespace : **jenkins**
        - Labels : **jenkins-slave**
      
      - **Add Container**를 클릭해서 Container 정보를 아래와 같이 입력합니다.    
        - Containers
          - Container Template (Build Job 실행을 위한 컨테이너)
            - Name : **jnlp**
            - Docker image : **jenkinsci/jnlp-slave**
            - Working directory : **/home/jenkins**
            
         <img width="1100" alt="Screen Shot 2020-04-12 at 8 08 29 PM" src="https://user-images.githubusercontent.com/6407492/79067309-69166e00-7cf9-11ea-914d-063ca99113ed.png">   
          
          - Container Template (Maven Build를 위한 컨테이너)
            - Name : **maven**
            - Docker image : **maven:3.3.9-jdk-8-alpine**
            - Working directory : **/home/jenkins**
            - Command to run : **cat**
          - Container Template (Docker Build & Push를 위한 컨테이너)
            - Name : **docker**
            - Docker image : **docker**
            - Working directory : **/home/jenkins**
            - Command to run : **cat** 
            
          <img width="603" alt="Screen Shot 2020-04-12 at 8 11 02 PM" src="https://user-images.githubusercontent.com/6407492/79067397-dd511180-7cf9-11ea-9a35-4aa9c5342dba.png"> 
              
          - Container Template (Kubectl 사용을 위한 컨테이너)
            - Name : **kubectl**
            - Docker image : **roffe/kubectl**
            - Working directory : **/home/jenkins**
            - Command to run : **cat** 
          - **Add Volume**를 클릭해서 Host Path Volume을 추가합니다.  
            - Host Path Volume
              - Host path : **/var/run/docker.sock**
              - Mount path : **/var/run/docker.sock**    
            
          <img width="628" alt="Screen Shot 2020-04-12 at 8 11 48 PM" src="https://user-images.githubusercontent.com/6407492/79067398-dfb36b80-7cf9-11ea-8c79-315968928e3d.png"> 
      
      - **Service Account**를 아래와 같이 설정합니다.  
        - Service Account : **jenkins**  
        
          <img width="873" alt="Screen Shot 2020-04-12 at 8 13 51 PM" src="https://user-images.githubusercontent.com/6407492/79067424-2608ca80-7cfa-11ea-9f50-369402562e41.png">  

---
### Table of Contents
[Amazon EKS에서 Jenkins로 분산 빌드 환경 구성하기](README.md)
1. [Jenkins 설치](100_install_jenkins.md)
2. [Jenkins 설정](200_configure_jenkins.md)
3. [빌드 작업 생성 및 실행](300_build_job_creation.md)
