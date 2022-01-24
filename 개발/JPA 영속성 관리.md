# JPA 영속성 관리  

[영속](#영속)  
[준영속](#준영속)  
## 영속성 컨텍스트(persistence context)
- 엔티티를 영구 저장하는 환경

## 엔티티의 생명주기
1. 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 상태
2. 영속(managed) : 영속성 컨텍스트에 저장된 상태
3. 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제(removed) : 삭제된 상태

### 비영속
```java
// 객체를 생성한 상태(비영속)
Member meber = new Member();
member.setId("member1");
member.setUsername("권씨");
```
![비영속](https://user-images.githubusercontent.com/71022555/150708331-79dd7666-3df8-457e-af71-afa57eba15a9.png)

### 영속
- 영속성 컨텍스트가 관리하는 엔티티를 영속 상태라 한다.
- persist() 뿐만 아니라 em.find(), JPQL을 사용해서 조회한 엔티티도 영속성 컨텍스트가 관리한다.
```java
em.persist(member);
```
![영속](https://user-images.githubusercontent.com/71022555/150708406-c53a2a84-17ef-49dd-a8be-f48a9602a7ef.png)
  
### 준영속
- 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다.
```java
em.detach(member);    // 특정 엔티티를 준영속 상태로 만들기
em.clear(); // 영속성 컨텍스트 초기화
em.close(); // 영속성 컨텍스트 닫기
```
  
### 삭제
```java
em.remove(member); // 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.
```

## 영속성 컨텍스트의 특징
1. 영속성 컨텍스트와 식별자 값
    - 영속성 컨텍스트는 엔티티를 식별자 값(@Id로 테이블의 기본 키와 매핑한 값)으로 구분한다. 따라서 영속 상태는 식별자 값이 반드시 있어야 한다.(없으면 예외 발생)
2. 영속성 컨텍스트와 데이터베이스에 저장
    - 영속성 컨텍스트에 엔티티를 저장하면 이 엔티티는 언제 데이터베이스에 저장되는 시점 -> 보통 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 DB에 반영하는데 이를 플러시(Flush)라 한다.
3. 영속성 컨텍스트가 엔티티를 관리하면 있는 장점
    - 1차 캐시
    - 동일성 보장
    - 트랜잭션을 지원하는 쓰기 지연
    - 변경 감지
    - 지연 로딩

## 엔티티 조회
- 영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이것을 1차 캐시라 한다. 영속 상태의 엔티티는 모두 이곳에 저장된다.
- 영속성 컨텍스트 내부에 Map이 하나 있는데 키는 @Id로 매핑한 식별자고 값은 엔티티 인스턴스다
```java
// 엔티티를 생성(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

// 영속
em.persist(member);
```
![1차캐시](https://user-images.githubusercontent.com/71022555/150708911-2b34aee3-aa9a-4b5f-8f49-c048dbf1e8c3.png)
```java
// 1차 캐시 조회
Member member = em.find(Member.class, "member1");
// 2차 캐시 조회
Member member2 = em.find(Member.class, "member2");
// find(엔티티 클래스 타입, 조회하는 엔티티 식별자 값)
// public<T> T find(class<T> entityClass, Object primaryKey);
```
- em.find() 호출 시 먼저 1차 캐시에서 엔티티를 찾고 만약 찾는 엔티티가 1차 캐시에 없으면 DB에서 조회
![1차캐시find](https://user-images.githubusercontent.com/71022555/150709086-f81a9968-f720-471c-be38-3b74d323c6be.png)
![db에서 조회](https://user-images.githubusercontent.com/71022555/150709144-ef422b63-b045-43c1-9f91-256496bce721.png)
### 동작과정
1. public<T> T find(class<T> entityClass, Object primaryKey); 를 실행한다.
2. 1차 캐시에 있으면 조회한 엔티티 반환 (없으면 DB에서 조회)
3. DB에서 조회한 데이터로 해당 이름(ex: member2) 엔티티를 생성해서 1차 캐시에 저장(영속)
4. 조회한 엔티티 반환

### 영속 엔티티의 동일성 보장
- 엔티티 조회 동작 과정을 보면 식별자가 같은 엔티티의 인스턴스는 1차 캐시에서 조회를 진행한다.
- 따라서 같은 엔티티 인스턴스를 반환하므로 동일성을 보장한다.
```java
Member test1 = em.find(Member.class, "admin");
Member test2 = em.find(Member.class, "admin");
System.out.println(test1==test2); -> return true // 동일성 비교시 참으로 나온다.
```


## 엔티티 등록
```java
//엔티티 매니저 팩토리 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

//엔티티 매니저 생성
EntityManager em = emf.createEntityManager(); 

//트랜잭션 기능 획득
EntityTransaction tx = em.getTransaction(); 
// 트랜잭션 시작
tx.begin();

em.persit(memberA);
em.persit(memberB);
// 여기까지 DB에 INSERT SQL을 데이터베이스에 보내지 않는다.

// 커밋하는 순간 DB에 INSERT SQL 전달
tx.commit(); // 트랜잭션 커밋
```
- 이처럼 트랜잭션을 커밋하기 직전까지 DB에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 차곡자곡 모아두며 커밋씨 쿼리를 DB에 전달하는데 이를 트랜잭션을 지원하는 쓰기 지연이라 한다.
![쓰기지연](https://user-images.githubusercontent.com/71022555/150710010-87e8d22b-ce5e-44bd-b3c5-7d43e742d529.png)
쓰기지연
![쓰기지연커밋](https://user-images.githubusercontent.com/71022555/150710022-43550a2c-393f-4d35-831d-57d39579e0ff.png)
쓰기지연커밋  

## 엔티티 수정
### SQL 수정 쿼리의 문제점
SQL을 사용하면 수정 쿼리를 직접 장성해야 한다. -> 유지 보수의 불편함 및 SQL 쿼리 분석 필수 -> 비지니스 로직이 SQL에 의존하게 된다.
```java
// 영속된 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

// 영속 엔티티 데이터 수정
memberA.setUsername("김씨");
memberA.setAge(22);

// em.update(member); 있어야 될 것 같지만 없는 메소드 -> 데이터를 변경만 해도 데이터베이스에 반영되기 때문

transaction.commit();

```
- 이처럼 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 변경 감지라 한다.
![변경감지](https://user-images.githubusercontent.com/71022555/150710330-8554eff3-46dc-4957-8209-ad23c2dcccce.png)
변경감지
- JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초의 상태를 복사해서 저장해두는데 이를 스냅샷이라 한다. 그리고 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다.
- 수정작업 작동 순서
    1. 트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 플러시(flush())가 호출된다.
    2. 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾는다.
    3. 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다.
    4. 쓰기 지연 저장소의 SQL을 DB에 보낸다.
    5. DB 트랜잭션을 커밋한다.
- 이처럼 변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다. -> 비영속은 변경해도 DB에 반영 X 
- 기본적으로 모든 필드를 UPDATE를 진행하므로 데이터 전송량이 증가한다. -> 동적 UPDATE SQL 전략 사용
```java
@org.hibernate.annotations.DynamicUpdate // 수정된 데이터만 사용해서 동적 UPDATE SQL 생성 대략 30over 시 효율 발생
@org.hibernate.annotations.DynamicInsert // 데이터를 저장할 때 데이터가 존재하는 필드(null이 아닌)만으로 INSERT SQL 동적으로 생성한다.
```
  
## 엔티티 삭제
엔티티를 삭제하려면 먼저 삭제 대상 엔티티를 조회해야 한다.
```java
Member memberA = em.find(Member.class, "memberA"); // 삭제 대상 엔티티 조회
em.remove(memberA); // 엔티티 삭제
```
- em.remove(엔티티)를 통해 쓰기 지연 SQL 저장소에 SQL 삭제 쿼리 등록
- em.remove(엔티티)를 호출시 해당 엔티티(ex: memberA)는 영속성 컨텍슽트에서 제거된다.
- 트랜잭션을 커밋해서 플러시를 호출하면 실제 데이터베이스에 삭제 쿼리 전달
  
## 플러시
플러시(flish())는 영속성 컨텍스트에 변경 내용을 DB에 반영한다. 플러시를 실행하면 구체적으로 다음과 같은 일이 일어난다.
1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다. 수정된 엔티티는 수정 쿼리를 만들어서 쓰기 지연 SQL 저장소에 등록한다.
2. 쓰기 지연 SQL 저장소의 쿼리를 DB에 전송한다(등록, 수정, 삭제 쿼리).  

- 영속성 컨텍스트를 플러시하는 방법
    1. em.flush()를 직접 호출한다.
    2. 트랜잭션 커밋 시 플러시가 자동 호출된다.
    3. JPQL 쿼리 실행 시 플러시가 자동 호출된다.
- 직접 호출
    - 엔티티 매니저의 flush() 메소드를 직접 호출해서 강제 플러시 ->테스트나 다른 프레임워크와 JPA를 함께 사용할 때를 제외하고 거의 사용하지 않는다.
- 트랜잭션 커밋 시 플러시 자동 호출
    - 트랜잭션을 커밋만 하면 어떠한 데이터도 DB에 반영되지 않으므로 JPA는 트랜잭션을 커밋하기전에 자동으로 플러시를 호출하여 영속성 컨텍스트의 변경 내용을 DB 반영한다.
- JPQL 쿼리 실행 시 플러시 자동 호출
    - JPQL은 SQL로 변환되어 DB에서 엔티티를 조회하므로 변경된 내용들을 반영해야 한다. 따라서 JPQL실행 전 플러시를 자동 호출한다.(but 식별자를 기준으로 조회하는 find()메소드는 플러시 실행 X)
```java
// JPQL 쿼리 실행 시 플러시 자동 호출 예제 코드
// 새로운 데이터들 저장한다고 가정
em.persist(member1);
em.persist(member2);
em.persist(member3);

result = em.createQuery("select m from Member m", Member.class);
List<Meber> members = query.getResultList(); // 조회 결과 저장
```

## 플러시 모드 옵션
엔티티 매니저에 플러시 모드를 직접 지정하려면 javax.persistence.FlushModeType을 사용하면 된다.
- FlushMode.Type.Auto : 커밋이나 쿼리 실행시 플러시(기본값)
-- FlushMode.Type.COMMIT : 커밋할 때만 플러시(성능 최적화를 위해 사용)
```java
em.setFlushMode(FlushModeType.COMMIT);
```

## 준영속
준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.
- 영속 상태의 엔티티를 준영속 상태로 만드는 방법
    1. em.detach(entity) : 특정 엔티티 준영속 상태로 전환.
    2. em.clear() : 영속성 컨텍스트를 완전히 초기화
    3. em.close() : 영속성 컨텍스트를 종료

### detach()
```java
public void detach(Object entity);
```
![detach](https://user-images.githubusercontent.com/71022555/150721606-98930e7d-08aa-45fb-aa33-131ed9441dcb.png)
  
### clear()
```java
em.clear();
```
em.detach()가 특정 엔티티를 준영속 상태로 만들었다면 em.clear()는 영속성 컨텍스트를 초기화해서 해당 영속성 컨텍스트의 모든 엔티티를 준 영속 상태로 만든다.
![영속성 초기화 전](https://user-images.githubusercontent.com/71022555/150721701-0da52d90-9e2a-4386-8b3c-cd916b8880d5.png) 초기화 전
![영속성 초기화 후](https://user-images.githubusercontent.com/71022555/150721725-39f6b463-06fb-49a5-80dc-458a28f9b575.png) 초기화 후
  
### close()
영속성 컨텍스트를 종료하여 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모둔 준영속 상태가 된다 detach() -> 내용을 초기화 close() -> 영속성 컨텍스트 자체를 종료
```java
em.close();
```
![영속성 종료 전](https://user-images.githubusercontent.com/71022555/150721936-e207a85d-1d42-4905-aae0-d4363d7a6d11.png)종료 전
![영속성 종료 후](https://user-images.githubusercontent.com/71022555/150721955-9cb5185f-8e10-4872-b56c-fff709f829c5.png)종료 후
  
## 준영속 상태의 특징
- 거의 비영속 상태에 가깝다
    - 영속성 컨텍스트가 관리하지 않으므로 1차 캐시, 쓰기 지연, 변경 감지, 지연 로딩을 포함한 영속성 컨텍스트가 제공하는 어떠한 기능도 동작하지 않는다.
- 식별자 값을 가지고 있다.
    - 비영속 상태는 식별자 값이 없을 수도 있지만 준영속 상태는 이미 한 번 영속 상태였으므로 반드시 식별자 값이 가지고 있다.
- 지연 로딩을 할 수 없다
    - 지연 로딩은 실체 객체 대신 프록시 객체를 로딩해주고 해당 객체를 실제 사용할 때 영속성 컨텍스트를 통해 데이터를 불러오는 방법이다. 하지만 준영속 상태는 영속성 컨텍스트가 더는 관리하지 않으므로 지연 로딩시 문제가 발생한다.
  
## 병합 : merge()
### 준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다. merge() 메소드는 준영속 상태의 엔티티를 받아서 그 정보로 새로운 영속 상태의 엔티티를 반환한다.
```java
public <T> T merge(T entity);
// ex)
Member mergeTest = em.merge(member);
```
![병합](https://user-images.githubusercontent.com/71022555/150747615-02720c16-2c7f-4896-b4e8-93e6859544e1.png)
  
### 비영속 엔티티도 영속 상태로 만들 수 있다.
```java
Member member = new Member();
Member newMember = em.merge(member); // 비영속 병합
tx.commit();
```
  
## 정리
- 엔티티 매니저는 엔티티 매니저 팩토리에서 생성한다. 자바를 직접 다루는 J2SE 환경에서는 엔티티 매니저를 만들면 그 내부에 영속성 컨텍스트도 함께 만들어진다. 이 영속성 컨텍스트는 엔티티 매니저를 통해서 접근할 ㅅ ㅜ있다.
- 영속성 컨텍스트는 애플리케이션과 DB 사이에서 객체를 보관한다. 영속석 컨텍스트 덕분에 1차 캐시, 동일성 보장, 트랜잭션을 지원하는 쓰기 지연, 변경 감지, 지연 로딩 기능을 사용할 수 있다.
- 영속성 컨텍스트에 저장한 엔티티는 플러시 시점에 DB에 반영되는데 일반적으로 트랜잭션을 커밋할 때 영속성 컨텍스트가 플러시된다.
- 영속성 컨텍스트가 관리하는 엔티티를 영속 상태의 엔티티라 하는데, 영속성 컨텍스트가 해당 엔티티를 더 이상 관리하지 못하면 그 엔티티는 준영속 상태의 엔티티라 한다. 따라서 준영속 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.
