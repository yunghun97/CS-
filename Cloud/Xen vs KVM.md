# 🥴 KVM vs Xen & OpenStack vs CloudStack


## 🌦 KVM(Kernel-based Virtual Machine)
- 리눅스 커널을 기반으로 만들어진 전가상화 오픈소스 하이퍼바이저입니다.
- Type-1(전가상화)으로 서버에 직접 하이퍼바이저를 설치하는 형식을 말합니다.
- Intel VT(Virtualization Technology) 또는 AMD-V 같은 가상화 기술을 지원하는 CPU가 있어야합니다.
- 

### Qemu
- Qemu는 가상화 솔루션이지만 Hypervisor 보다는 에뮬레이터라고 말하는 것이 정확한 표현이다.
- KVM-Qemu는 대부분 같이 사용을 하는데 가상화 + 에뮬레이션의 효과를 얻기 위해 사용한다고 예상 됨 OR HVM을 위해?(향후 추가 예정)
- 일반적인 성능은 에뮬레이터가 가상화보다는 성능이 낮으므로 Qemu < KVM


## ☁ Xen  
- 오픈소스 하이퍼바이저
- 초기에는 대표적인 반가상화 하이퍼바이저 였지만 **HVM(하드웨어 기반 가상화)** 기능을 활용하여 전가상화도 지원한다.
  
![image](https://user-images.githubusercontent.com/71022555/190321017-673d5bcf-aeee-49c2-83fa-dfb96978a545.png)  
  
- 도메인(Domain)은 Xen 환경에서 동작하고 있는 가상머신을 의미합니다.  
    - Dom0 : 실제 물리 디바이스와 통신하기 위한 디바이스 드라이브가 존재하는 가상화된 시스템이라고 보면 된다. Hypervisor 위에서 작동하는 VM으로 가상화를 위한 Host가 동작하는 환경을 의미
    - Dom0(Domain0) : 하이퍼바이저를 제어하고 다른 게스트 운영 체제를 시작하는 역할을 합니다.
    - DomU(Domain Unit) : 특권이 없는(Unprivileged)(하드웨어에 접근할 수 없는) 나머지 도메인 DomU라고 한다. 게스트 시스템(OS)가 동작하는 것
      - PVM Guest : Linux와 같이 가상화에 맞게 수정된 OS
      - HVM Guest : 주로 Windows OS 경우를 말하며 수정이 불가능한 OS로 PV 드라이버 설치 필요
       
### 동작 원리
1. 각 도메인(DomU)은 Hypercall로 Xen 하이퍼바이저에게 입/출력 요청을 하게 되고, Xen 하이퍼바이저는 이를 Dom0에게 전달한다.
2. Dom0는 받은 요청을 실제 Device에 전달하고 이런 방식으로 실제 장치와 입/출력을 하게 된다.





## ⛈ OpenStack
```
클라우드 환경에 대한 모든 타입을 지원하는 오픈소스 클라우드 컴퓨팅 플랫폼이다.  
Openstack은 서로 보완하는 다양한 서비스를 통하여(Iaas) 솔루션을 제공합니다.  
Application Programming Interface(API)를 이욯여 각 서비스 통합을 쉽게 구성합니다.  
일반 가상화 기술과의 차이점은 오픈소스이며, 물리적인 자원의 성능을 최대한 활용하면서 동시에 클라이언트의 니즈에 맞춰 부하 분산을 할 수 있다.

```
```

```

### 장점
1. 경제성
  - 무료로 사용 가능하다(Apache 2.0 license)
2. 신뢰성
  - 개발, 사용 기간이 10년 남짓
3. 중립성
  - 오픈소스 환경이기 떄문에 다른 Vendor사 없이 독립적으로 사용가능

### 단점
1. 복잡성
  - 해당 지식을 잘 알고 있는 IT 직원이 필요하다.
2. 지원
  - 다른 회사나 팀이 소유한 기술이 아니므로 오픈소스 커뮤니티에 지원을 의존해야 한다.
3. 일관성
  - Openstack 구성 요소 제품군은 구성 요소가 추가, 삭제 될 때 항상 유동적입니다.


## 🌩 CloudStack
### 온 디맨드 방식으로, 유연한 클라우드 컴퓨팅 서비스를 설정


## OpenStack vs CloudStack

|분류|OpenStack|CloudStack|
|:---:|:---:|:---:|
|개발언어|Python|주(Java) 보조(Python)|
|설치 시간|느림(1~3일)|빠름(2시간)|
|문서화|비교적 별로|비교적 나음|
|커뮤니티|많음|적음|
|기능|비교적 많음|적음|
|사용자 편의성|별로|좋음|

### KT CLOUD(기준)

### D1(OpenStack)
- scale up, scale out 둘 다 지원
- OpenStack 기반 IOT, 빅데이터, 인메모리 분석 등 고급 기술 지원 및 도입을 위한 가상화 환경을 제공
- 월 추가 비용이 필요함(30만/월)


### G1, G2(Cloud Stack)
- scale out만 지원
  
|기능|D1(OpenStack)|G1, G2(CloudStack)|
|:---:|:---:|:---:|
|Server|O|O|
|High Memory Server|O|O|
|인프라 부가서비스|O|O|
|Data Lake|O|X|
|AI Studio|O|X|
|DevOps Suite|O|O|
|Container|O|X|


##### 참고자료
##### https://suyeon96.tistory.com/54
##### https://blog.naver.com/alice_k106/221179347223
##### https://www.joinc.co.kr/w/Site/cloud/Qemu/Basic  

