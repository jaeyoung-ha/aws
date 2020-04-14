### 1. Jenkins 설치

Amazon EKS에 Jenkins를 설치합니다.  
EBS볼륨을 정적 또는 동적으로 바인딩해서 Jenkins를 설치합니다.    
EBS CSI Driver를 사용해서 Storage Class, Persistent Volume, Persistent Volume Claim을 생성합니다.    
Jenkins를 위한 Deployment, Service, Ingress를 배포합니다.   

Jenkins를 설치하기 전에, 준비작업 필요합니다.  

- 본 실습에 필요한 파일들을 github repository에서 clone합니다.

  ```
  git clone https://github.com/openzon/jenkins-on-eks.git
  ```

- 아래 링크의 가이드를 참조해서 **EBS CSI Driver를 배포합니다.**  
  - [Amazon EBS CSI Driver 배포하기](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/ebs-csi.html)    


준비 작업을 완료하였고, Kubernetes 리소스를 배포해서 Jenkins를 설치합니다.   

- Jenkins Namespace를 생성합니다.

  ```bash
  kubectl create namespace jenkins
  ```
  
- Service Account, Role, RoleBinding, ClusterRole, ClusterRoleBinding를 생성합니다.

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl -n jenkins apply -f jenkins/jenkins.rbac.yaml
  ```
  
  - jenkins.rbac.yaml
  
    - Service Account
    
      ```yaml
      ---
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: jenkins
      ```
    
    - Role
    
      ```yaml
      ---
      kind: Role
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
        name: jenkins
      rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: ["extensions", "apps"]
        resources: ["deployments"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: [""]
        resources: ["pods/exec"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: [""]
        resources: ["pods/log"]
        verbs: ["get","list","watch"]
      - apiGroups: [""]
        resources: ["secrets"]
        verbs: ["get"]
      ```
    
    - RoleBinding
    
      ```yaml
      ---
      apiVersion: rbac.authorization.k8s.io/v1
      kind: RoleBinding
      metadata:
        name: jenkins
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: Role
        name: jenkins
      subjects:
      - kind: ServiceAccount
        name: jenkins
        namespace: jenkins
      ```
    
    - ClusterRole
    
      ```yaml
      ---
      kind: ClusterRole
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
        name: jenkins
      rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: ["extensions", "apps"]
        resources: ["deployments"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: [""]
        resources: ["pods/exec"]
        verbs: ["create","delete","get","list","patch","update","watch"]
      - apiGroups: [""]
        resources: ["pods/log"]
        verbs: ["get","list","watch"]
      - apiGroups: [""]
        resources: ["secrets"]
        verbs: ["get"]
      ```
    
    - ClusterRoleBinding
    
      ```yaml
      ---
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: jenkins
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: jenkins
      subjects:
      - kind: ServiceAccount
        name: jenkins
        namespace: jenkins  
      ```

- **Static Volume을 사용할 경우,** EBS Volume을 생성합니다. 또는 사용할 기존 Volume을 선택합니다.

  ```bash
  aws ec2 create-volume --availability-zone=ap-northeast-2a --size=50 --volume-type=gp2 
  ```
  
  - EBS Volume 생성 결과는 아래와 비슷합니다. Persistent Volume을 생성할 때, VolumeId가 필요합니다.
  
    ```json
    {
        "AvailabilityZone": "ap-northeast-2a",
        "CreateTime": "2020-04-13T04:15:07.000Z",
        "Encrypted": false,
        "Size": 50,
        "SnapshotId": "",
        "State": "creating",
        "VolumeId": "vol-0ab4695bb1e3227be",
        "Iops": 150,
        "Tags": [],
        "VolumeType": "gp2",
        "MultiAttachEnabled": false
    }
    ```
    
- **Dynamic Volume을 사용할 경우,** ***이 단계를 건너뛰고 Storage Class 생성 단계로 넘어갑니다.***

- Storage Class를 생성합니다.

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl apply -f jenkins/jenkins.sc.yaml
  ```
  
  - jenkins.sc.yaml  
  
    ```yaml
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: jenkins
    provisioner: ebs.csi.aws.com
    volumeBindingMode: WaitForFirstConsumer
    reclaimPolicy: Retain
    ```
  
- **Static Volume을 사용할 경우,** Persistent Volume을 생성합니다.  

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl apply -f jenkins/jenkins.pv.yaml
  ```
  
  - jenkins.pv.yaml (***volumeHandle를 여러분의 EBS VolumeId로 변경해야 합니다.***)
  
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: jenkins
    spec:
      capacity:
        storage: 50Gi
      volumeMode: Filesystem
      accessModes:
        - ReadWriteOnce
      storageClassName: jenkins
      persistentVolumeReclaimPolicy: Retain
      csi:
        driver: ebs.csi.aws.com
        volumeHandle: vol-0ab4695bb1e3227be
        fsType: ext4
      nodeAffinity:
        required:
          nodeSelectorTerms:
          - matchExpressions:
            - key: topology.ebs.csi.aws.com/zone
              operator: In
              values:
              - ap-northeast-2a   
    ```
    
- **Dynamic Volume을 사용할 경우,** ***이 단계를 건너뛰고 Persistent Volume Claim 생성 단계로 넘어갑니다.*** 

- Persistent Volume Claim를 생성합니다.

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl -n jenkins apply -f jenkins/jenkins.pvc.yaml
  ```
  
  - jenkins.pvc.yaml
  
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: jenkins-claim
    spec:
      storageClassName: jenkins
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 50Gi
    ```
    
- Jenkins Deployment를 배포합니다.

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl -n jenkins apply -f jenkins/jenkins.deployment.yaml
  ```
  
  - jenkins.deployment.yaml
  
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: jenkins
      labels:
        name: jenkins
        app: jenkins
    spec:
      replicas: 1
      selector:
        matchLabels:
          name: jenkins
      template:
        metadata:
          labels:
            app: jenkins
            name: jenkins
          name: jenkins
        spec:
          serviceAccountName: jenkins
          containers:
          - env:
            - name: JAVA_OPTS
              value: -Xmx2048m -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
            image: jenkins/jenkins #:lts-alpine 
            imagePullPolicy: IfNotPresent
            name: jenkins
            ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 50000
              protocol: TCP
            # resources:
            #   limits:
            #     cpu: "1"
            #     memory: 1Gi
            #   requests:
            #     cpu: "1"
            #     memory: 1Gi
            volumeMounts:
            - mountPath: /var/jenkins_home
              name: jenkins
          restartPolicy: Always
          securityContext:
            #fsGroup: 1000
            runAsUser: 0
          terminationGracePeriodSeconds: 30
          volumes:
          - name: jenkins
            persistentVolumeClaim:
              claimName: jenkins-claim  
    ```
    
- Jenkins Service/Ingress를 배포합니다.

  ```bash
  cd ~/environment/jenkins-on-eks
  kubectl -n jenkins apply -f jenkins/jenkins.service.yaml
  ```
  
  - jenkins.service.yaml
  
    - Service
    
      ```yaml
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: jenkins
        labels:
          app: jenkins
      spec:
        type: NodePort
        ports:
          - name: ui
            port: 8080
            targetPort: 8080
            protocol: TCP
          - name: slave
            port: 50000
            protocol: TCP
          - name: http
            port: 80
            targetPort: 8080
        selector:
          app: jenkins
      ```
    
    - Ingress
    
      ```yaml
        ---
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: jenkins
          namespace: jenkins
          annotations:
            kubernetes.io/ingress.class: alb
            alb.ingress.kubernetes.io/scheme: internet-facing
          labels:
            app: jenkins
        spec:
          rules:
            - http:
                paths:
                  - path: /*
                    backend:
                      serviceName: jenkins
                      servicePort: 80  
      ```
    
---
### Table of Contents
[Jenkins on Amazon EKS](README.md)
1. [Jenkins 설치](100_install_jenkins.md)
2. [Jenkins 설정](200_configure_jenkins.md)
3. [빌드 작업 생성 및 실행](300_build_job_creation.md)
