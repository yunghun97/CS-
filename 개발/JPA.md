# JPA
- 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다.
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다.  


### JPA 설정
```
spring.datasource.url=jdbc:h2:tcp://localhost~~
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.show-sql=true  -> jpa가 날리는 sql 표시
spring.jpa.hibernate.ddl-auto=none -> 자동으로 테이블 생성x 
```
### spring.jpa.hibernate.ddl-auto
- none: No change to the database structure.
- update: Hibernate changes the database according to the given Entity structures. - 변경된 스키마만 적용한다.
- validate: 변경된 스키마가 있는지 확인만 한다. 만약 변경이 있다면 Application을 종료한다.
- create: Creates the database every time, but don’t drop it when close. - 시작될 때만 drop하고 다시 생성한다.
- create-drop: Creates the database then drops it when the SessionFactory closes. - 시작과 종료에서 모두 drop한다.

[JPA CRUD](#JPA-CRUD)  
[상속](#상속)  


## JPA CRUD
### 저장 기능
### jpa.persist('객체');
- persist() 메소드는 객체를 데이터베이스에 저장한다.
- 이 메소드를 호출하면 JPA가 객체와 매핑정보를 보고 적절한 INSERT SQL을 생성해서 데이터베이스에 전달한다.  

### 조회 기능
``` java
String memberId = "testId";
Member member = jpa.find(Member.class, memberId); // 조회
```
- find() 메소드는 객체 하나를 데이터베이스에서 조회한다.
- JPA는 객체와 매핑정보를 보고 적절한 SELECT SQL을 생성해서 데이터베이스에 전달하고 그 결과로 Member 객체를 생성해서 반환한다.  

## 수정 기능 
```java
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경"); // 수정
```
- JPA는 별도의 수정 메소드를 제공하지 않는다.
- 대신 객체를 조회해서 값을 변경만 하면 트랜잭션을 커밋할 때 데이트베이스에서 적절한 UPDATE SQL이 전달된다.  

## 연관된 객체 조회
```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```
- JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다.
- 따라서 JPA를 사용하면 연관된 객체를 마음껏 조회할 수 있다.

## 상속
테이블 형태
![image](https://user-images.githubusercontent.com/71022555/149879001-b16e60f1-4212-400f-b0f1-226ec282e82a.png)

1. jpa.persist(album)
2. JPA는 다음 SQL을 실행해서 객체를 ITEM, ALBUM 두 테이블에 나누어 저장한다.
    - INSERT INTO ITEM ...
    - INSERT INTO ALBUM ...
3. Album 객체를 조회 find() 
    - String albumId = "id1";
    - Album album = jpa.find(Album.class, albumId)l
    - SELECT I.\*, A.\* FROM ITEM I
    JOIN ALBUM ON I.ITEM_ID = A.ITEM_ID
  
## 연관관계
- 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고 조인을 사용해서 연관된 테이블을 조회한다.
- 객체는 참조가 있는 방향으로만 조회할 수 있다.  

테이블에 맞춘 객체 모델
```java
class Member{
    String id;  // Member_id
    Long teamId; // TEAM_ID FK 컬럼
    String username;    
}
class Team{
    Long id;    // TEAM_ID PK
    String name;
}
```
참조를 사용하는 객체 모델
```java
class Member{
    String id;  // Member_id
    Team team; // 참조로 연관관계를 맺는데
    String username;
    Team getTeam(){
        return team;
    }
}
class Team{
    Long id;    // TEAM_ID PK
    String name;
}
```
## 저장
- team 필드를 TEAM_ID 외래 키 값으로 변환해야 한다.
- 다음처럼 외래 키 값을 찾아서 INESRT SQL을 만들어야 할 때 Member 테이블에 저장해야 할 TEAM_ID 외래 키는 TEAM 테이블의 기본키 이므로 member.getTeam().getId()로 구할 수 있다.
## 조회
- TEAM_ID 외래 키 값을 Member 객체의 team 참조로 변환해서 객체에 보관해야 한다.
- SQL : SELECT M.\*, T.\* FROM MEMBER M 
JOIN TEAM T ON M.TEAM_ID = T>TEAM_ID
개발자가 직접 연관관계 설정
```java
public Member find(String memberId){
    // SQL 실행
    Member member = new Member();

    // 데이터베이스에서 조회한 회원 관련 정보를 모두 입력
    Team team = new Team();
    
    // 데이터 베이스에서 조회한 팀 관련 정보를 모두 입력

    // 회원과 팀 관계 설정
    member.setTeam(team);
    return member;
}
```
이러한 과정들은 모두 패러다임 불일치를 해결하려고 소모하는 비용이다.
만약 자바 컬렉션에 회원 객체를 저장한다면 이런 비용이 전혀 들지 않는다.

## JPA와 연관관계
- JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다.
코드 예시
```java
member.setTeam(team); // 회원과 팀 연관관계 설정
jpa.persist(member); // 회원과 연관관계 함께 저장
```
- 개발자는 회원과 팀의 관계를 설정하고 회원 객체를 저장하면 된다.
- JPA는 team의 참조를 외래 키로 변환해서 적절한 INSERT SQL을 데이터베이스에 전달한다.
- 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.
예시 코드
```java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
```
  
## 객체 그래프 탐색
연관 관계 가정
![객체 연관관계](https://user-images.githubusercontent.com/71022555/149882130-20ba98a7-800c-4b24-ace6-875fe9cec101.png)
객체 그래프 탐색 코드
```java
member.getOrder().getOrderItem()...
// member.getOrder() = null 일 경우 더 탐색할 수 없다.
```
- SQL에서는 어디까지 탐색할 수 있을지 정해지지만 객체지향 개발자에게는 제약이 된다.
- Cuz 비지니스 로직에 따라 사용하는 객체 그래프가 다른데 언제 끊어질지 모를 객체 그래프를 함부로 탐색할 수는 없기 때문이다.
```java
// 상황에 따라 모든 메소드를 만들어서 사용 -> 비효율적
memberDAO.getMember();  // Member만 조회
memberDAO.getMemberWithTeam();  // Member와 Team조회
memberDAO.getMemberWithOrderWithDelivery(); // Member, Order, Delivery조회
```
## JPA와 객체 그래프 탐색
- JPA를 사용하면 객체 그래프를 마음껏 탐색할 수 있다.
- JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행하기 때문이다.
- 이 기능은 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 해여 **지연 로딩**이라 한다.
- JPA는 지연 로딩을 투명하게 처리한다.
```java
// 투명한 엔티티
class Member{
    private Order order;
    public Order getOrder(){    // 어떤 코드도 직접 사용하지 않는다.
        return order;
    }
}
```
```java
// 지연 로딩을 사용하는 코드

// 처음 조회 시점에 SELECT MEMBER SQL 실행
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); // Order를 사용하는 시점에 SELECT ORDER SQL
```
- Meber를 사용할 때마다 Order를 함께 사용하면 이렇게 한 테이블씩 조회하는것보다 Member를 조회하는 시점에 SQL 조인을 사용해서 Member와 Order를 함께 조회하는 것이 효과적이다.
- JPA는 연관된 객체를 즉시 함께 조회할지 아니면 실제 사용되는 시점에 지연해서 조회할지 간단한 설정으로 정의할 수 있다.  

# 비교

<!-- ### ORM (Object, Relational, Mapping)
- annotation으로 mapping한다.
```java
@Entity
public class Member{
    
    //pk mapping
    @Id  
    // db가 자동으로 생성하주는거 = IDENTITY
    @GeneratedValue(strategy = GeneratedType.IDENTITY)
    private long id;
    
    // db에 컬럼명과 일치시킬 때 @Column(name = "username")
    private String name;

    public long getId() {
        return id;
    }
    public void setId(long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```
org.springframework.boot:spring-boot-starter-data-jpa 설정 시
### EntityManager 스프링 부트에서 생성 -> Injection 사용하면 된다.
- 내부적으로 DataSource를 통해 DB와 통신한다.
```java
private final EntityMananger em;
em.persist(member); // jpa가 insert 쿼리 및 setId 등을 해준다.

Member member = em.find(Member.class, id); -> select 결과 반환

List<Member> result =  em.createQuery("select m from Member m", Member.class).getResultList(); // select m 인 경우 Member 자체를 select 한다는 뜻

m.createQuery("select m from Member where m.name = :name", Member.class).setParameter("name",name).getResultList();
``` -->