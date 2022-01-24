
[JPA CRUD](#JPA-CRUD)  
[상속](#상속)  
[JPA와 연관관계](#JPA와-연관관계)  
[JPA를 사용해야 하는 이유](#JPA를-사용해야-하는-이유)  
[매핑](#매핑)  
[JPQL](#JPQL)

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

```java
jpa.perist(album); // ITEM, ALBUM 두 테이블에 나누어서 정보 저장
String albumId = "id97";
Album album = jap.find(Album.class, albumId);

// ITEM과 ALBUM 두 테이블을 조인해서 필요한 데이터를 조회하고 그 결과를 반환한다.
SELECT I.\*, A.\* FROM ITEM I
JOIN ALBUM ON I.ITEM_ID = A.ITEM_ID
```
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

## 비교
- 데이터베이스는 기본 키의 값으로 각 fow를 구분한다.
- 반면 객체는 동일성(identity)비교와 동등성(equality)비교라는 두 가지 비교 방법이 있다.
    - 동일성 비교는 == 비교다. 객체 인스턴스의 주소 값을 비교한다.
    - 동동성 비교는 equals() 메소드를 사용하여 객체 내부의 값을 비교한다.
```java
class MemberDAO{
    public Member getMember(String memberId){
        String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
        // JDBC API, SQL 실행
        return new Member(...);
    }
}

String memberId = "50";
Member member1 = memberDAO.getMemeber(memberId);
Member member2 = memberDAO.getMemeber(memberId);

member1 == member2; // 동일성 비교 -> 서로 다르다.

Member member1 = list.get(0);
Member member2 = list.get(0);

member1 == member2; // 동일성 비교 -> 같다 -> 객체를 컬렉션에 보관
```
## JPA와 비교
- JPA는 같은 트랙잭션일 때 같은 객체가 조회되는 것을 보장한다.
```java
String memberId = "50";
Member member1 = memberDAO.getMemeber(memberId);
Member member2 = memberDAO.getMemeber(memberId);

member1 == member2; // 동일성 비교 -> JPA에서는 같다.
```

## 정리
- ### 정교한 객체 모델링을 할수록 패러다임의 불일치 문제가 더 커진다.
- ### 그 틈을 메우기 위해 개발자가 소모해야 하는 비용이 많이지므로 객체 모델링 중심에서 데이터 중심의 모델로 변해간다.
- ### JPA는 패러다임의 불일치 문제를 해결해주고 정교한 객체 모델링을 유지하게 도와줍니다.
  

# JPA
- ### 자바 진영의 ORM 기술 표준으로 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- ### JPA를 사용하면, SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환을 할 수 있다.
- ### JPA를 사용하면 개발 생산성을 크게 높일 수 있다.  
![JPA 동작 방식](https://user-images.githubusercontent.com/71022555/149893183-7786030b-d91b-4d16-8582-e688ef898401.png)
애플리케이션과 JDBC 사이에서 동작한다.
- JPA 1.0(2006) : 복합 키와 연관관계 기능이 부족
- JPA 2.0(2009) : 대부분 ORM 기능을 포함하고 JPA Criteria가 추가
- JPA 2.1(2013) : 스토어드 프로시저 접근, 컨버터, 엔티티 그래프 기능이 추가
# ORM
- ### 객체와 관계형 데이터베이스를 매핑한다는 뜻
- ### ORM 프레임워크는 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다.
- ### ex) INSERT SQL을 직접 작성하는 것이 아니라 객체를 마차 자바 컬렉션에 저장하듯이 ORM 프레임워크에 저장하면 ORM 프레임워크가 적절한 INSERT SQL을 생성해서 데이터베이스에 객체를 저장해준다.  
- ### 객체와 관계형 데이터베이스를 어떻게 매핑할 것인지 ORM 프레임워크에게 알려주면 된다.
- ### 주로 하이버네이트 프레임워크 사용
```java
jpa.persist(member); // 저장
```
![JPA 저장](https://user-images.githubusercontent.com/71022555/149894597-5458dbcc-6ca2-41f2-94d6-4f4d1d5e6343.png)
```java
Member member = jpa.find(memberId); // 조회
```
![JPA 조회](https://user-images.githubusercontent.com/71022555/149894442-3f2f1a30-e914-4882-9d9f-682d0a14e0e6.png)  

## JPA를 사용해야 하는 이유
### 1. 생산성
    - JPA를 사용하면 자바 컬렉션에 객체를 저장하듯이 JPA에게 저장할 객체를 전달하면 된다.
    - 데이터베이스 설계 중심의 패러다임을 객체 설계 중심으로 역전시킬 수 있다.
### 2. 유지보수
    - SQL 을 직접 다루면 엔티티에 필요한 필드를 하나만 추가해도 관련 등록, 수정, 조회 SQL과 결과를 매핑하는 JDBC API를 모두 수정해야 했다.
    - JPA를 사용하면 이런 과정을 JPA가 대신 처리해주므로 필드를 추가하거나 삭제해도 수정해야 할 코드가 줄어든다.
### 3. 패러다임의 불일치 해결
    - 상속, 연관관계, 객체 그래프 탐색, 비교하기와 같은 패러다임의 불일치 문제를 해결해준다.
### 4. 성능
    - JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.
    - 이렇게 애플리케이션과 데이터베이스 사이에 계층이 존재하면 최적화 관점에서 시도 해 볼 수 있는 것들이 많다.
```java
String MemberId = "helloId";
Member member1 = jpa.find(memberId);
Member member2 = jpa.find(memberId);
// JPA를 사용하면 회원을 조회하는 SELECT SQL을 한 번만 데이터베이스에 전달하고 두 번째는 조회한 회원 객체를 사용한다.
// 히어비네이트는 SQL 힌트를 넣을 수 있는 기능도 제공
```
### 5. 데이터 접근 추상화와 벤더 독립성
    - 관계형 데이터베이스는 같은 기능도 벤더마다 사용법이 다른 경우가 많다.
    - JPA는 데이터베이스 변경시 JPA에게 다른 데이터베이스를 사용한다고 알려주기만 하면 된다.
    - ex) 로컬 개발 환경은 H2 DB를 사용, 개발이나 사용은 Oracle이나 MySQL DB를 사용할 수 있다.
### 6. 표준
    - JPA는 자바 진영의 ORM 기술 표준이다. 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다.

# 개발
## 매핑
### @Entity
- 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다.
- 이렇게 @Entity가 사용된 클래스를 엔티티 클래스라 한다.
### @Table
- 엔티티 클래스에 매핑할 테이블 정보를 알려준다.
 name 속성을 사용해서 매핑한다.
 - 어노테이션 생략시 클래스 이름을 테이블 이름으로 매핑한다.
### @Id
- 엔티티 클래스의 필드를 테이블의 기본 키에 매핑한다.
- 이렇게 @Id가 사용된 필드를 식별자 필드라 한다.
### @Column
- 필드를 컬럼에 매핑한다. name 속성을 사용해서 매핑한다.
### 매핑 정보가 없는 필드
- 매핑 어노테이션을 생략하면 필드명을 사용해서 매핑한다.
```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private int age
}
```
### persistence.xml
```html
<!--- XML 네임 스페이스와 사용할 버전을 지정한다.-->
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
    <!--영속성 유닛 설정 (고유 이름을 부여해야 한다)-->
    <persistence-unit name="jpabook">    
        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>
```

## 기본 동작
```java
        //엔티티 매니저 팩토리 생성 -> META-INF/persistence.xml에서 이름이 jpabook인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

        //엔티티 매니저 생성
        EntityManager em = emf.createEntityManager(); 

        //트랜잭션 기능 획득
        EntityTransaction tx = em.getTransaction(); 

        try {


            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료

        public static void logic(EntityManager em) {

        String id = "yunghun97";
        Member member = new Member();
        member.setId(id);   // update와 같다.
        member.setUsername("권씨");
        member.setAge(26);

        //등록
        em.persist(member);

        //수정
        member.setAge(27);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);

        }
```
### 엔티티 매니저 생성 과정
![엔티티 매니저 생성 과정](https://user-images.githubusercontent.com/71022555/150483875-d797e58f-afaf-42d1-8e3b-21d78931f4ec.png)

1. persistence.xml의 설정 정보를 사용해서 엔티티 매니저 팩토리를 생성한다. -> 이때 Persistence 클래스를 사용해서 엔티티 매니저 팩토리를 생성해서 JPA를 사용할 수 있도록 준비한다.
2. 엔티티 매니저 팩토리를 생성할 때 데이터베이스 커넥션 풀도 생성하므로 팩토리를 생성하는 비용이 크다 -> 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용한다.
3. 엔티티 매니저 팩토리에서 엔티티 매니저 생성 -> 매니저에서 JPA 기능을 대부분 제공.
4. 엔티티 매니저는 내부에 데이터소스(DB Connection)을 유지하며 데이터베이스와 통신 -> 가상의 데이터 베이스라고 생각하면 된다. (DB Connection과 밀접한 관계가 있으므로 스레드가 공유 재사용 불가)
5. 엔티티 매니저 종료
6. 엔티티 팩토리 매니저 종료

## 트랜잭션 관리
- JPA는 트랜잭션 안에서 데이터를 변경해야 한다.(안 그러면 예외 발생)
- 엔티티 매니저에서 트랜잭션 API를 받아와서 사용한다.
  
## JPQL
- JPA는 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색한다.
- 검색조건이 포함된 SQL을 사용하는데 이를 JPQL이라는 쿼리 언어로 해결한다.
- JPQL : 엔티티 객체를 대상으로 쿼리한다. 즉 클래스와 필드를 대상으로 쿼리한다.
- SQL : 데이터베이스 테이블을 대상으로 쿼리한다.
- JPQL은 대소문자를 구분한다.
```SQL
select m from Member m
/* from Member은 Member 테이블이 아니라 엔티티 객체를 말하는 것이다.
```
### JPQL 사용
1. em.createQuery(JPQL, 반환타입) 메소드를 실행해서 쿼리 객체를 작성한 후
2. 쿼리객체의 getResultList 메소드를 호출하면 된다.
ex)
```java
em.createQuery("select m from Member m", Member.class).getResultList();
// 이를 JPA가 JPQL을 분석해서 SELECT M.ID, M.NAME, M.AGE FROM MEMBER M 으로 SQL 생성 후 데이터베이스에서 데이터를 조회한다.
```









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