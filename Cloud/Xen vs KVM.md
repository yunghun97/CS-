# 🥴 Xen vs KVM

## ☁ Xen

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
|설치 시간|느림(1~3일)|빠름(2시간)|
|문서화|비교적 별로|비교적 나음|
|커뮤니티|많음|적음|
|기능|비교적 많음|적음|
|사용자 편의성|별로|좋음|

### KT CLOUD(기준)

### G1, G2(Cloud Stack)
- scale out만 지원

### D1(OpenStack)
- scale up, scale out 둘 다 지원
- OpenStack 기반 IOT, 빅데이터, 인메모리 분석 등 고급 기술 지원 및 도입을 위한 가상화 환경을 제공
- 월 추가 비용이 필요함(30만/월)
  
|기능|D1(OpenStack)|G1, G2(CloudStack)|
|:---:|:---:|:---:|
|Server|O|O|
|High Memory Server|O|O|
|인프라 부가서비스|O|O|
|Data Lake|O|O|
|AI Studio|O|X|
|DevOps Suite|O|O|
|Container|O|X|
