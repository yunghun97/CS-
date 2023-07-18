# 🎥Kubernates
**컨테이너 가상화** 기술을 서비스간에 자원격리를 하는데 OS를 별도로 안띄워도 된다.  
**OS 기동시간이 없기 때문에** 자동화시에 엄청 빠르고, 자원 효율도 매우 높습니다.
> 엄청 많은 컨테이너, 서비스를 운영 시 이를 일일이 배포하고 운영하는 역할을 해주는게 **컨테이너 오케스트레이터**라는 개념이다. (여러 컨테이너를 관리해주는 솔루션)  
많은 클라우드 서비스 기업들이 쿠버네티스 환경이 설치되어 있는 인프라를 서비스 하고 있다.

# 기술 스택
## USER
- Workloads
    - Pod
    - Replication Controller
    - ReplicaSet
    - StatefulSet
    - Deployment
    - DaemonSet
    - Job
    - CronJob
- Service
    - Service
    - Endpoints
    - Ingress
- Storage / Config
    - PV
    - PVC
    - ConfigMap
    - Secret
    - StorageClass
    - Volume
- Metadata
    - LimitRange
    - CRD
    - Event
    - HPA
    - MWC / VWC
    - PriorityClass
    - PodTemplate
    - PodDisruption
    - PodPreset
    - PodSecurity Policy
- Cluster
    - Namespace
    - ResourceQuota
    - Role / RoleBinding
    - Policy
    - ServiceAcoount
    - NodeScheduling
    - ...
## Admin
- Installation
- Architecture
- Networking
- Monitoring
- Plugins
- Ecosystem

## Kubernetes를 사용해야 하는 이유?
- **Auto Scaling**에 따라 트래픽양에 따라 서버 자원을 바꿔준다.
- **Auto Healing** 장애가 발생한 서버위에 있는 서비스들이 다른 자원으로 바꿔서 할당해준다.
- **Deployment** 기능에 따라 효율적인 배포 가능
  
## VM vs Container
도커는 여러 컨테이너들간의 호스트 자원을 분리해서 사용하게 해준다. 이것은 리눅스 고유기술인 name space와 cgroup 를 사용하여 격리하는것이다.
- namespace : 커널에 관련 된 영역을 분리
    - mnt, pid, net, ipc, uts, user
- cgroups : 자원에 대한 영역을 분리
    - memory, CPU, I/O, network

![image](https://user-images.githubusercontent.com/71022555/165774963-294d9d54-b628-4dec-849e-0e58e021da85.png)
  
|Virtual Machine|Container|
|:---|:---|
|각각의 OS를 공유|한 OS를 공유|
|리눅스 VM에서 Window 컨테이너 사용 가능|리눅스 컨테이너에서 Window용 컨테이너 사용 X|
|보안적 측면 유리|보안적 측면 불리|
|원하는 Guest OS를 통해 작동|필요한 서비스가 돌아가는데 필요한 라이브러리가 있는 이미지당 만들어서 동작|