# 😀 JPA 다양한 연관관계 매핑

## 🍕 다대일
> 다대일 관계의 반대 방향은 항상 일대다 관계고 일대다 관계의 반대 방향은 항상 다대일 관계이다.  
> **데이터베이스 테이블의 1:N 관계에서 외래 키는 항상 N 쪽에 있다.**  
> 따라서 객체 양방향 관계에서 연관관계의 주인은 항상 N(다)쪽이다. 회원(N)과 팀(1)이 있으면 회원쪽이 연관관계의 주인이다.

### 다대일 단방향[N:1]  
![image](https://user-images.githubusercontent.com/71022555/174429565-befdc610-05b1-4d51-9373-75e8412c892a.png)  

```java
// 회원
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;

}
// 팀
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    private String naeme;
}
```

### 다대일 양방향[N:1, 1:N]  
![image](https://user-images.githubusercontent.com/71022555/174435503-9915c2b1-e659-46e5-b328-f6d0b1ea432d.png)  
```java
// 회원
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    public void setTeam(Team team){
        this.team = team;
        // 무한루프 빠지지 않도록 체크
        if(!team.getMembers.contains(this)){
            team.getMembers.add(this);
        }
    }
}

// 팀
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy="team") // mappedBy가 있으므로 주인이 아님
    private List<Member> members = new ArrayList<Member>();

    public void addMember(Member member){
        this.members.add(member);
        // 무한루프 빠지지 않도록 체크
        if(member.getTeam() != this){
            member.setTeam(this);
        }
    }
}
```
- 양방향은 외래 키가 있는 쪽이 연관관계의 주인이다.
> 일대다와 다대일 연관관계는 항상 다(N)에 외래키가 있다. 여기서는 다쪽인 Member 테이블이 외래 키를 가지고 있으므로 Member.team이 연관관계의 주인이다. JPA는 외래 키를 관리할 때 연관관계의 주인만 사용한다. 주인아 아닌 Team.members는 조회를 위한 JPQL이나 객체 그래프를 탐색할 때 사용한다.
- 양방향 연관관계는 항상 서로를 참조해야 한다.
> 어느 한 쪽만 참조하면 양방향 연관관계가 성립하지 않는다. 항상 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋은데 회원의 setTeam(), 팀의 addMember()가 이런 편의 메소드들이다. 편의 메소드는 양쪽에 다 작성할 수도 있는다. 양쪽 작성 시 무한루프에 빠지지 않도록 검사해야 한다.
## 🍔 일대다
- 다대일 관계의 반대 방향이다. 일대다 관계는 엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인 List, Set, Map중에 하나를 사용해야 한다.
### 일대다 단방향[1:N]
- 하나의 팀은 여러워 회원을 참조할 수 있는데 이런 관계를 일대다 관계라 한다. 
- 팀은 회원들은 참조하지만 반대로 회원은 팀을 참조하지 않으면 둘의 관계는 단방향이다.  

![image](https://user-images.githubusercontent.com/71022555/174436156-a6f100e9-5fa9-460c-b63c-5acfab350b59.png)  

> 일대다 단방향 관계는 팀 엔티티의 Team.members로 회원 테이블의 TEAM_ID 외래 키를 관리한다. 보통 자신이 매핑한 테이블의 외래 키를 관리하는데 이 매핑은 반대쪽 테이블에 있는 외래 키를 관리한다.  
> 그럴 수 밖에 없는 것이 일대다 관계에서 외래 키는 항상 다쪽 테이블에 있다. 하지만 다쪽인 Member 엔티티에는 외래 키를 매핑할 수 있는 참조 필드가 없다. 대신 반대쪽인 TEAM에만 참조 필드인 members가 있다. 따라서 반대편 테이블의 외래 키를 관리하는 특이한 모습이 나타난다.
```java
// 팀
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    private String name;

    @OneToMany
    @JoinColumn(name="TEAM_ID") // Member 테이블의 TEAM_ID(FK)
    private List<Member> members = new ArrayList<Member>();
}

// 회원
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;
    
}
```
> 일대다 단방향 관계를 매핑할 때는 @JoinColumn을 명시해야 한다. 그렇지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본을 사용해서 매핑한다.

### 일대다 단방향 매핑의 단점
- 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점이다.
- 본인 테이블에 외래 키가 있으면 엔티티의 저장과 연관관계 처리를 INSERT SQL 한 번으로 끝낼 수 있지만 다른 테이블에 왜래 키가 있으면 연관관계 처리를 위한 UPDATE SQL을 추가로 실행해야 한다.
```java
public void saveTest(){
    Member member1 = new Member("member1");
    Member member2 = new Member("member2");

    Team team1 = new Team("team1");
    team1.getMembers().add(member1);
    team1.getMembers().add(member2);

    em.persist(member1); // INSERT member1
    em.persist(member2); // INSERT member2
    em.persist(team1) // INSERT team1,  UPPDATE=member1.fk, UPDATE=member2.fk
}
```
```sql
-- 위 코드 SQL 실행 결과
insert into Member(MEMBER_ID, username) values(null, ?)
insert into Member(MEMBER_ID, username) values (null, ?)
insert into Team(TEAM_ID, name) values(null, ?)
update Member set TEAM_ID=? where MEMBER_ID=?
update Member set TEAM_ID=? where MEMBER_ID=?
```
- Member 엔티티는 Team 엔티티를 모른다. 그리고 연관관계에 대한 정보는 Team 엔티티의 members가 관리한다. 따라서 Member 엔티티를 저장할 때는 MEMBER 테이블의 TEAM_ID 외래 키에 아무 값도 저장되지 않는다. 대신 TEAM 엔티티를 저장할 때 TEAM.members의 참조 값을 확인해서 회원 테이블이 TEAM_ID를 업데이트 한다.
> 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자.  
  
> 일대다 단방향 매핑을 사용하면 엔티티를 매핑한 테이블이 아닌 다른 테이블의 외래 키를 관리해야 한다. 이는 성능뿐만 아니라 관리도 부담스럽다. 이를 해결하는 쉬운 방법은 다대일 양방향 매핑을 사용하는 것이다. 다대일 양방향 매핑은 관리해야 하는 외래 키가 본인 테이블에 있다. 

### 일대다 양방향[1:N, N:1]
- 일대다 양방향 매핑은 존재하지 않는다.
- 대산 다대일 양방향 매핑을 사용해야 한다.(똑같은 말이다. 여기서는 왼쪽을 연관관계의 주인으로 가정해서 분류 ex) 다대일이면 다(N)이 연관관계의 주인이다.)
- **정확히 말하자면 양방향 매핑에서 @OneToMany는 연관관계의 주인이 될 수 없다.** 
- 관계형 데이터베이스의 특성 상 다대일 관계는 항상 다(N)쪽에 외래 키가 있다. 
- 따라서 @OneToMany, @ManyToOne 둘 중에 연관관계의 주인은 항상 다 쪽인 @ManyToOne을 사용한 곳이다. 이런 이유로 @ManyToOne에는 mappedBy 속성이 없다.
- 완전히 불가능한 것은 아니다. 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 다대일 단뱡항 매핑을 읽기 전용으로 하나 추가하면 된다.  

![image](https://user-images.githubusercontent.com/71022555/174436649-568afeb9-8946-4de2-8c40-e06e93052d4d.png)  
  
```java
@Entity
public class Team{
    @Id @GeneratedValue
    @Column(name="TEAM_ID")
    private Long id;
    private String name;

    @OneToMany
    @JoinColumn(name="TEAM_ID")
    private List<Member> members = new ArrayList<Member>();
}

// 회원
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;
    
    @ManyToOne
    @JoinColumn(name="TEAM_ID", insertable = false, updatable = false)
    private Team team;
}
```
- 다대일 쪽은 insertable updatable = false 설정을 추가해서 읽기만 가능하게
- 일대다 양방향 매핑이라기 보다는 일대다 단방향 매핑 반대편에 다대일 단뱡향 매핑을 읽기 전용으로 추가해서 일대다 양방향처럼 보이도록 하는 방법이다.
- 일대다 단방향 매핑이 가지는 단점을 그대로 가진다.
- 될 수 있으면 다대일 양뱡항 매핑을 사용하자

## 🍟 일대일[1:1]
- 양쪽이 서로 하나의 관계만 가진다.
- 회원은 하나의 사물함만 사용하고 사물함도 마찬가지
- 다음과 같은 특징이 있다.
    - 일대일 관계는 그 반대도 일대일 관계이다.
    - 테이블 관계에서 일대다, 다대일은 항상 다(N)쪽이 외래 키를 가진다. 반면에 일대일 관계는 어느 곳이나 외래 키를 가질 수 있다.
- 주 테이블에 왜래 키
    - 주 테이블에 외래 키를 두고 대상 테이블을 참조한다. 외래 키를 객체 참조와 비슷하게 사용할 수 있어서 객체지향 개발자들이 선호한다. 이 방법은 **주 테이블이 외래 키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있다.
- 대상 테이블에 외래 키
    - 전통적인 DB 개발자들은 보통 대상 테이블에 외래 키를 두는 것을 선호한다. 이 방법의 장점은 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.

### 주 테이블에 외래 키
- 객체지향 개발자들이 선호한다.
- JPA도 주 테이블에 외래 키가 있으면 좀 더 편리하게 매핑할 수 있다.
### 단방향
- MEMBER가 주 테이블, LOCKER는 대상 테이블  
![image](https://user-images.githubusercontent.com/71022555/174436954-409026dc-22cb-4190-ad6d-009996c9d65a.png)  
```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;
}
@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;
}
```
- 1:1 관계이므로 @OneToOne 사용
### 양방향  

![image](https://user-images.githubusercontent.com/71022555/174437026-a13fbb59-4657-4de1-8050-f3f477b4d70c.png)  


```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name="LOCKER_ID")
    private Locker locker;
}
@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;
    @OneToOne(mappedBy="locker")
    private Member member;
}
```
- Member.locker가 연관관계의 주인이다.
- Locker.member는 mappedBy를 통해 연관관계의 주인이 아니라고 설정했다.

### 대상 테이블에 외래 키
#### 단방향  
![image](https://user-images.githubusercontent.com/71022555/174437109-4682c997-0c87-45b7-94b4-fefadeb99a87.png)  
  
- 그림을 보면 대상 테이블에 외래 키가 있는 단방향 관계는 JPA에서 지원하지 않는다. 그리고 이런 모양으로 매핑할 수 있는 방법도 없다.
- 이때는 단방향 관계를 Locker에서 Member 방향으로 수정하거나, 양방향 관계로 만들고 Locker를 연관관계의 주인으로 설정해야 한다.
- JPA2.0 부터 일대다 단방향 관계에서 대상 테이블에 외래 키가 있는 매핑을 허용했다. 하지만 일대일 단방향은 이런 매핑을 허용하지 않는다.
  
#### 양방향  
![image](https://user-images.githubusercontent.com/71022555/174437166-c5119f90-6f37-4f7a-af4f-ecc9183ef70f.png)  

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    private String username;

    @OneToOne(mappedBy="member")
    private Locker locker;
}
@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;
}
```
  
> 프록시를 사용할 때 외래 키를 관리하지 않는 일대일 관계는 지연 로딩으로 설정해도 즉시 로딩된다. 예를 들어 Locker.member는 지연로딩 할 수 있지만 Member.locker는 지연로딩으로 설정해도 즉시 로딩된다. 이것은 프록시 한계 때문에 발생하는 문제로 **bytecode instrumentation을 사용하면 해결할 수 있다.**
  
## 🍿 다대다[N:N]
- 관계형 데이터 베이스는 정규화된 테이블 2개로 다대다 관계를 **표현할 수 없다.**
- 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.  
  
![image](https://user-images.githubusercontent.com/71022555/174437407-f37ca950-70ae-4e99-8079-469acaf2ac88.png)  

- 아래 그림처럼 중간에 연결 테이블을 추가해야한다.
- Member_Product 연결 테이블을 추가해서 다대다 관계를 일대다, 다대일 관계로 풀어낼 수 있다.
- 이 연결 테이블은 회원이 주문한 상품을 나타낸다.  

![image](https://user-images.githubusercontent.com/71022555/174437434-d85b4d05-0ae2-4914-8d5e-d68773769eef.png)  
  
> 그런데 객체는 테이블과 다르게 객체 2개로 다대다 관계를 만들 수 있다. 예를 들어 회원과 상품 객체는 사로 컬렉션을 사용해여 참조하면 된다. @ManyToMany를 사용  

![image](https://user-images.githubusercontent.com/71022555/174437527-37aa76b1-335c-493a-862d-7fbc827bd9db.png)  
  
```java
// 다대다 단방향 회원
@Entity
public class Member{
    @Id @Column(name="MEMBER_ID")
    private String id;
    private String username;

    @ManyToMany
    @JoinTable(name="MEMBER_PRODUCT", joinColumns = @JoinColumn(name="MEMBER_ID"),
    inverseJoinColumns = @JoinColumn(name="PRODUT_ID"))
    private List<Product> products = new ArrayList<Product>();

}
//  다대다 단방향 상품
@Entity
public class Product{
    @Id @Column(name="PRODUCT_ID")
    private String id;
    private String name;
}
```
- 회원 엔티티와 상품 엔티티를 @ManyToMany로 매핑했다. **중요한 점은 @ManyToMany와 @JoinTable을 사용해서 연결 테이블을 바로 매핑한 것이다. 따라서 회원과 상품을 연결하는 회원_상품(Member_Product) 엔티티 없이 매핑을 완료할 수 있다.**
- 연결 테이블을 매핑하는 **@JoinTable의 속성**
    - **@JoinTable.name** : 연결 테이블을 지정한다. 여기서는 MEMBER_PRODUCT 테이블을 선택
    - **@JoinTable.joinColumns** : 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다. MEMBER_ID로 지정
    - **@JoinTable.inserseJoinColumns** : 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다. PRODUCT_ID로 지정했다.
- MEMBER_PRODUCT 테이블은 다대다 관계를 일대다, 다대일 관계로 풀어내가 위한 연결 테이블일 뿐이다. @ManyToMany로 매핑한 덕분에 다대다 관계를 사용할 때는 이 연결 테이블을 신경 쓰지 않다도 된다.
  
```java
//  다대다 관계를 저장
public void save(){
    Product productA = new Product();
    productA.setId("productA");
    productA.setName("상품A");
    em.persist(productA);

    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    member1.getProducts().add(ProductA); // 연관관계 설정
    em.empersist(member1);
}
```
```sql
-- 결과
INSERT INTO PRODUCT ...
INSERT INTO MEMBER ...
INSERT INTO MEMBER_PRODUCT ...
```
```java
//  다대다 탐색
public void find(){
    Member member = em.find(Member.class, "member1");
    List<Product> products = nmember.getProducts(); // 객체 그래프 탐색
    for(Product product : products){
        System.out.println("product.name = " + product.getName());
    }
}
```
```sql
-- member.getProducts()
SELECT * FROM MEMBER_PRODUCT MP
INNER JOIN PRODUCT P ON MP.PRODUCT_ID=P.PRODUCT_ID
WHERE MP.MEMBER_ID =?
```
- SQL을 보면 MEMBER_PRODUCT와 상품 테이블을 조인해서 연관된 상품을 조회한다.
- 복잡한 다대다 관계를 애플리케이션에서는 아주 단순하게 사용할 수 있다.

### 다대다 양방향
- 역방향도 @ManyToMany를 사용한다. 그리고 원하는 곳에 mappedBy로 연관관계의 주인을 지정한다.
```java
@Entity
public class Product{
    @Id
    private String id;
    @ManyToMany(mappedBy="products") // 역방향 추가
    private List<Member> members;
}
```
> 다대다의 양방향 연관관계는 다음처럼 설정하면 된다.  
> **member.getProducts().add(product);**  
> **product.getMembers().add(member);**
- 양방향 연관관계는 연관관계 편의 메소드를 추가해서 관리하는 것이 편리하다.
```java
public void addProduct(Product product){
    products.add(product);
    product.getMembers().add(this);
}
```
#### 역방향 탐색
```java
public void findInverse(){
    Product product = em.find(Product.class, "productA");
    Lsit<Member> members = product.getMembers();
    for(Member member : members){
        System.out.println("member = "+member.getUsername());
    }
}
```

## 🥚 다대다 : 매핑의 한계와 극복, 연결 엔티티 사용
- @ManyToMany를 실무에서 사용하기에는 한계가 있다.
- ex) 회원이 상품을 주문하면 연결 테이블에 단순히 주문한 회원 아이디와 상품 아이디만 담고 끝나지 않는다.
- 보통은 연결 테이블에 주문 수량 컬럼이나 주문한 날짜 같은 컬럼이 더 필요하다.  
  
![image](https://user-images.githubusercontent.com/71022555/174438183-dbcb29d6-7da6-4054-b8a4-40bd2efae362.png)  
  
- 이렇게 컬럼을 추가하면 @ManyToMany를 사용할 수 없다.
  
![image](https://user-images.githubusercontent.com/71022555/174601057-3bea4ac8-7cd2-43e9-a466-9f78846ac43b.png)
  
- 연결 테이블을 매핑하는 엔티티를 만들고 이곳에 추가한 컬럼들을 매핑해야 한다.
- 테이블의 관계도 테이블 관계처럼 다대다에서 일대다, 다대일 관계로 풀어야한다.

```java
// 다대다 회원
@Entity
public class Member{
    @Id @Column(name="MEMBER_ID")
    private String id;
    // 역방향
    @OneToMany(mappedBy="member")
    private List<MemberProduct> memberProducts;

}
// 회원과 회원상품을 양방향 관계로 만들었다. 회원상품 엔티티쪽이 외래 키를 가지고 있으므로 연관관계의 주인이다. 따라서 연관관계의 주인이 아닌 회원의 Member.memberProducts에는 mappedBy를 사용했다,

//  다대다 상품
@Entity
public class Product{
    @Id @Column(name="PRODUCT_ID")
    private String id;
    private String name;
}
// 상품 엔티티에서 회원상품 엔티티로 객체 그래프 탐색 기능이 필요하지 않다고 판단해서 연관관계를 만들지 않았다.

// 회원상품 엔티티
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct{

    @Id
    @ManyToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member; // MemberProductId.member와 연결
    @Id
    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product; // MemberProductId.product와 연결
    private int orderAmount;
}

// 회원상품 식별자 클래스
public class MemberProductId implements Serializable{
    private String member; // MemberProduct.member와 연결
    private String product; // MemberProduct.product와 연결

    // hashCode and equals

    @Override
    public boolean equals(Object O){...}

    @Override
    public int hashCode() {...}
}
```
> 회원상품(MemberProduct) 엔티티를 보면 기본 키를 매핑하는 @Id와 외래 키를 매핑하는 @JoinColumn을 동시에 사용해서 기본 키 + 외래 키를 한번에 매핑했다. 그리고 @IdClass를 사용해서 복합 기본 키를 매핑했다.

### 🎂 **복합 기본 키**
- 회원상품 엔티티는 기본 키가 MEMBER_ID와 PRODUCT_ID로 이루어진 복합 기본키(간단히 복합키)다. JPA에서 복합 키를 사용하려면 별도의 식별자 클래스를 만들어야 한다.
- 그리고 엔티티에 @IdClass를 사용해서 식별자 클래스를 지정하면 된다. 여기서는 MemberProductId 클래스를 복합 키를 위한 식별자 클래스로 사용한다.
- **복합 키를 위한 식별자 클래스는 다음과 같은 특징이 있다.**
    - 복합 키는 별도의 식별자 클래스로 만들어야 한다.
    - Serializable을 구현해야 한다.
    - equals와 hashCode 메소드를 구현해야 한다.
    - 기본 생성자가 있어야 한다.
    - 식별자 클래스는 pulic이어야 한다.
    - @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있다.
### 식별 관계
- 회원상품은 회원과 상품의 기본 키를 받아서 자신의 기본 키로 사용한다.
- 이렇게 부모 테이블의 기본 키를 받아서 자신의 기본 키 + 외래 키로 사용하는 것을 데이터베이스 용어로 식별 관게(Identifying Relationship)라 한다.
- **회원상품(MemberProduct)은 회원의 기본 키를 받아서 자신의 기본 키로 사용함과 동시에 회원과의 관계를 위한 외래 키로 사용한다. 그리고 상품의 기본 키도 받아서 자신의 기본 키로 사용함과 동시에 상품과의 관계를 위한 외래 키로 사용한다. 또한 MemberProductId 식별자 클래스로 두 기본 키를 묶어서 복합 기본 키로 사용한다.**

> 사용방법_저장
```java
public void save(){
    // 회원 저장
    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    em.persist(member1);

    // 상품 저장
    Product productA = new Product();
    productA.setId("productA");
    productA.setName("상품1");
    em.persist(productA);

    // 회원상품 저장
    MemberProduct memberProduct = new MemberProduct();
    memberProduct.setMember("member1"); // 주문 회원 - 연관관계 설정 
    memberProduct.setProduct(productA); // 주문 상품 - 연관관계 설정
    memberProduct.setOrderAmount(2); // 주문 수량

    em.persist(memberProduct);
}
```
- 회원상품 엔티티를 만들면서 연관된 회원 엔티티와 상품 엔티티를 설정했다.
- 회원상품 엔티티는 데이터베이스에 저장될 때 연관된 회원의 식별자와 상품의 식별자를 가져와서 자신의 기본 키 값으로 사용한다.
  
> 사용방법_조회
```java
public void find(){
    // 기본 키 값 생성
    MemberProductId memberProductId = new MemberProduct();
    memberProductId.setMember("member1");
    memberProductId.setProduct(productA); 

    MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);

    Member member = memberProduct.getMember();
    Product product = memberProduct.getProduct();

}
```
- 복합 키는 항상 식별자 클래스를 만들어야 한다. em.find()를 보면 생성한 식별자 클래스로 엔티티를 조회한다.
- 복합 키를 사용하는 방법은 복잡하다. 단순히 컬럼 하나만 기본 키로 사용하는 것과 비교해서 복합 키를 사용하면 ORM 매핑에서 처리할 일이 상당히 많아진다.
- 복합 키를 위한 식별자 클래스 생성과 @IdClass 또는 @EmbeddedId도 사용해야 한다. 그리고 식별자 클래스에 equals, hashCode도 구현해야 한다.

### 🥂 다대다: 새로운 기본 키 사용
  
![image](https://user-images.githubusercontent.com/71022555/174605064-35df1d9b-5d07-4a96-9ebf-d58f3dc4ee2e.png)  
  
- 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것을 추천한다.
    - 간편하고 거의 영구히 쓸 수 있으며 비즈니스에 의존하지 않는다.
    - ORM 매핑 시에 복합 키를 만들지 않아도 되므로 간단히 매핑을 완성할 수 있다.
- 위 그림은 ORDER_ID라는 새로운 기본 키를 하나 만들고 MEMBER_ID, PRODUCT_ID 컬럼은 외래 키로만 사용한다.

```java
// 주문(ORDER 테이블) 코드
@Entity
public class Order{
    @Id @GeneratedValue
    @Column(name="ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name="MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name="PRODUCT_ID")
    private Product product;

    private int orderAmount;

}
```
- 대리 키를 사용함으로써 이전에 보았던 식별 관계에 복합 키를 사용하는 것보다 매핑이 단순하고 이해하기 쉽다.
- 회원 엔티티와 상품 엔티티는 변경사항이 없다.
```java
// 회원
@Entity
public class Member{
    @Id @Column(name="MEMBER_ID")
    private String id;
    // 역방향
    @OneToMany(mappedBy="member")
    private List<MemberProduct> memberProducts;

}

//  상품
@Entity
public class Product{
    @Id @Column(name="PRODUCT_ID")
    private String id;
    private String name;
}
```
사용방법_저장
```java
public void save(){
    // 회원 저장
    Member member1 = new Member();
    member1.setId("member1");
    member1.setUsername("회원1");
    em.persist(member1);

    // 상품 저장
    Product productA = new Product();
    productA.setId("productA");
    productA.setName("상품1");
    em.persist(productA);

    // 주문 저장
    Order order = new Order();
    order.setMember(member1); // 주문 회원 - 연관관계 설정
    order.setProduct(productA); // 주문 상품 - 연관관계 설정
    order.setOrderAmount(2); // 주문 수량
    em.persist(order);
}
```
사용방법_조회
```java
public void find(){
    Long orderId = 1L;
    Order order = em.find(Order.class, orderId);

    Member member = order.getMember();
    Product product = order.getProduct();
}

```

### 🌮 다대다 연관관계 정리
> 다대다 관계를 일대다 다대일 관계로 풀어내기 위해 연결 테이블을 만들 때 식별자를 어떻게 구성할지 선택해야 한다.
1. 식별 관계 : 받아온 식별자를 기본 키 + 외래 키로 사용한다.
2. 비식별 관계 : 받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가한다.
- DB 설계에서는 1번처럼 부모 테이블의 기본 키를 받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 것을 식별 관계라 하고 2번처럼 단순히 외래키로만 사용하는 것을 비식별 관계라 한다.
- 객체 입장에서 보면 2번처럼 비식별 관계를 사용하는 것이 복합 키를 위한 식별자 클래스를 만들지 않아도 되므로 단순하고 편리하게 ORM 매핑을 할 수 있다. 이런 이유로 2번 비식별 관계를 추천함

### 정리
> 이번에는 다대일, 일대다, 일대일, 다대다 연관관계를 단방향, 양방향으로 매핑하는 방법에 대해 알아보았다. 마지막에는 다대다 연관관계를 일대다, 다대일 연관관계로 풀어보았다.

참고자료
---
##### 자바 ORM 표준 JPA 프로그래밍