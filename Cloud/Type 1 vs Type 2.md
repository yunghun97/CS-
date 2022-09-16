# 🌩 Type 1 vs Type 2

## 1️⃣ Type 1  
```
가상화 소프트웨어가 OS의 형태(실직적인 OS는 아니지만 바이오스에 기능이 추가된 형태)
```
![image](https://user-images.githubusercontent.com/71022555/190531635-23dfacea-8e8c-40d0-859d-202757a3aaad.png)  
- 대표적인 소프트웨어로 Xen와 Hyper-V가 있다.
- 주로 서버 개발에 많이 사용된다.
- **장점**
  - Hypervisor가 하드웨어 통제권을 가지고 있다는 장점이 있다.
  - Type 2는 Hypervisor가 하드웨어 통제권이 없어 하드웨어를 제어하는데 오버헤드가 많이 걸렸지만 Type1은 장악을 하고있으며 이를 Guest OS에 빠르게 전달하기 위한 API도 있다.
  - 이때 구현되는 Guest OS는 Para-virtualization(반가상화)의 형태로 구현된다.  
  - 
**개발 시 요구사항**
1. 신속성 : Hypervisor는 Guest OS와 Hardware사이의 중간 관리자이며 Guest OS가 요구하는 정보들을 빠르게 전달해야한다.
2. Guest OS 수정 : Guest OS와 Hypervisor가 통신이 가능하도록 코드(커널) 수정
3. 안정성 : Guest OS 동작이 안정적으로 동작하도록 해야한다.
  
 ## 2️⃣ Type 2
 ```
 가상화 기술을 애플리케이션의 형태로 구현하는 방식
 ```
 - 대표적인 예로는 VMware, Virtual Box처럼 윈도우나 리눅스에 설치 할 수 있는 소프트웨어이다.  
 ![image](https://user-images.githubusercontent.com/71022555/190532231-9f941262-5845-43cf-bb65-a746008d495b.png)  
   
  - 이러한 구현 방식은 기존 PC 시장에 쉽게 가상화 기술 소프트웨어를 도입 할 수 있다는 이점이 있다.
  - 또한 Host OS가 제공하는 하드웨어 제어권을 빌려오는 형태여서 여러가지 요소들을 신경쓰지 않고 구현 가능
  - 애플리케이션 형태이므로 안정성에 문제가 생겨도 Host OS에 영향을 미치지 않는다.
  - **동작 속도는 느리다.**
    - 소프트웨어가 하드웨어를 직접 통제하는 것이 아니기 때문에 하드웨어에 명령을 전달하거나 받을 때 Host OS를 거쳐야 하므로 느리다.
    - 
 


 



##### 출처
##### https://selfish-developer.com/entry/%EA%B0%80%EC%83%81%ED%99%94-%EA%B8%B0%EC%88%A0-Type1?category=825819
##### https://selfish-developer.com/entry/%EA%B0%80%EC%83%81%ED%99%94%EA%B8%B0%EC%88%A0-Type2?category=825819
