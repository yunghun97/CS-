# 🧙‍♀️ @PersistenceContext


## EntityManager를 빈으로 주입할 때 사용하는 어노테이션입니다.
- 스프링에서는 영속성 관리를 위해 EntityManager가 존재합니다.
- 스프링 컨테이너가 시작될 때 EntityManager를 만들어서 빈으로 등록해둡니다.
- 이 때 스프링이 만들어둔 EntityManager를 주입받을 때 사용합니다.


## @PersistenceContext로 지정된 프로퍼티에 아래 두 가지 중 한 가지로 EntityManager를 주입해줍니다.
1. EntityManagerFactory에서 새로운 EntityManager를 생성하거나
2. Transaction에 의해 기존에 생성된 EntityManager를 반환해줍니다.
  
--- 
## @PersistenceContext를 사용해야 하는 이유
- **EntityManager를 사용할 때 주의해야 할 점은 여러 쓰레드가 동시에 접근하면 동시성 문제가 발생하여 쓰레드 간에는 EntityManager를 공유해서는 안됩니다.**
- 일반적으로 스프링은 싱글톤 기반으로 동작하기에 빈은 모든 쓰레드가 공유합니다.
- **그러나 @PersistenceContext으로 EntityManager를 주입받아도 동시성 문제가 발생하지 않습니다.**

### 동시성 문제가 발생하지 않는 이유는
- 스프링 컨테이너가 초기화되면서 @PersistenceContext으로 주입받은 EntityManager를 Proxy로 감쌉니다.
- 그리고 EntityManager 호출 시 마다 Proxy를 통해 EntityManager를 생성하여 Thread-Safe를 보장합니다.


##### 출처
##### https://batory.tistory.com/497
