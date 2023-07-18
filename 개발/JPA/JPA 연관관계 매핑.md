# 🐼 JPA 연관관계 매핑

### 방향
- 단방향 : 회원 -> 팀
- 양방향 : 회원 -> 팀, 팀 -> 회원  
> 방향은 객체관계에만 존재하고 테이블 관계는 항상 양방향이다.
  
### 다중성
- N:1, 1:N, 1:1, N:N
- 회원 -> 팀 :  회원과 팀(다대일) 팀과 회원(일대다) 
  
### 연관관계의 주인
객체를 양방향 연관관계로 만들면 연관관계의 주인을 정해야한다.
  
## 단방향 연관관계
### 연관관계 중에선 (N:1) 단방향 관계를 가장 먼저 이해해야 한다. 지금부터 회원과 팀의 관계를 통해 다대일 단방향 관계를 알아보자
- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계이다.
![image](https://user-images.githubusercontent.com/71022555/164226235-4b73dfaa-f9e2-4389-bdab-a4dea88de475.png) 
##### 그림1
  
### 그림 설명
- 객체 연관관계
    - 회원 객체는 Member.team 필드(멤버 변수)로 팀 객체와 연관관계를 맺는다.
    - 회원 객체와 팀 객체는 단방향 관계이다. 회원은 Member.team 필드를 통해서 팀을 알 수 있지만 반대로 팀은 회원을 알 수 없다. 예를 들어 member -> team의 조회는 member.getTeam()으로 가능하지만 반대 방향인 team -> member를 접근하는 필드는 없다.
- 테이블 연관관계
    - 회원 테이블은 TEAM_ID 외래 키로 팀 테이블과 연관관계를 맺는다.
    - 회원 테이블과 팀 테이블은 양방향 관계이다. 회원 테이블의 TEAM_ID 외래 키를 통해서 회원과 팀을 조인할 수 있고 반대로 팀과 회원도 조인할 수 있다. 예를 들어 MEMBER 테이블의 TEAM_ID 외래 키 하나로 MEMBER JOIN TEAM과 TEAM JOIN MEMEBER 둘 다 가능하다.
- 객체 연관관계와 테이블 연관관계의 가장 큰 차이
> 참조를 통한 연관관계는 언제나 **단방향**이다. 객체간에 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다. 결국 연관관계를 하나 더 만들어야 한다.  이를 **양방향 연관관계라 한다.** 정확하게는 **서로 다른 단 방향 관계 2개이다.**
```java
// 단방향 연관관계
class A{
    B b;
}
class B{

}
// 양방향 연관관계
class A{
    B b;
}
class B{
    A a;
}
```
### 객체 연관관계 vs 테이블 연관관계
> 객체는 **참조(주소)**로 연관관계를 맺는다.  
테이블은 **외래 키**로 연관관계를 맺는다.  
이 둘은 비슷해 보이지만 객체는 참조(a.getB())를 사용하지만 테이블은 **조인**을 사용한다.
- 참조를 사용하는 객체의 연관관계는 단방향이다.
- 외래키를 사용하는 테이블의 연관관계는 양방향이다.

## 🤠 객체 관계 매핑
위의 그림1 기준 JPA를 사용하여 둘을 매핑해보자.
```java
@Entity
public class Member{
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;

    // 연관관계 설정
    public void setTeam(Team team){
        this.team = team;
    }
}

@Entity
public class Team{
    @Id
    @Coluimn(name ="TEAM_ID")
    private String id;
    
    private String name;
}
```
- 객체 연관관계 : 회원 객체의 Member.team을 사용
- 테이블 연관관계 : 회원 테이블의 MEMBER.TEAM_ID 외래 키 컬럼 사용
### 어노테이션 설명
> **@ManyToOne** : 이름 그대로 다대일(N:1) 관계라는 매핑 정보다. 회원과 팀은 다대일 관계이다. 연관관계를 매핑할 때 이렇게 다중성을 나타내는 어노테이션을 필수로 사용해야 한다.  
**@JoinColumn(name="TEAM_ID")** : 조인 컬럼은 외래 키를 매핑할 때 사용한다. name 속성에는 매핑할 외래 키 이름을 지정한다. 회원과 팀 테이블은 TEAM_ID 외래 키로 연관관계를 맺으므로 이 값을 지정하면 된다. 이 어노테이션은 생략할 수 있다.

### @JoinColumn
|속성|기능|기본값|
|:---|:---|:---|
|name|매핑할 외래 키 이름|필드명+_+참조하는 테이블의 기본 키 컬럼 명|
|referencedColumnName|외래 키가 참조하는 대상 테이블의 컬럼명|참조하는 테이블의 기본 키 컬럼명|
|foreignKey(DDL)|외래 키 제약조건을 직접 지정할 수 있다. 이 속성은 테이블을 생성할 때만 사용한다.||
|unique, nullable, insertable, updatable, columnDefinitaion, table|@Column 속성과 같다.||

### @ManyToOne
|속성|기능|기본값|
|:---|:---|:---|
|optional|false로 설정하면 연관된 엔티티가 항상 있어햐 한다.|true|
|fetch|글로벌 패치 전략을 설정한다.(N+1) 문제|@ManyToOne=FetchType.EAGER @OneToMany=FetchType.LAZY|
|cascade|영속성 전이 기능을 사용한다.||
|targetEntity|연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제너릭으로 타입정보를 알 수 있다.||
```java
// targetEntity 설명
// 제너릭
@OneToMany
private List<Member> members;

@OneToMany(targetEntity=Member.class)
private List members;
```

## 🐑 연관관계 사용
### 저장
```java
public void test(){
    // 팀1 저장
    Team team1 = new Team("team1", "1팀");
    em.persist(team1); // save()는 저장후 ID 값을 return한다. persist는 반환형이 void

    //회원 저장
    Memeber member1 = new Member("member1", "회원1");
    member1.setTeam(team); // 연관관계 설정 memeber1 -> 
    em.persist(member1);
    team1
}
```
### 조회
연관관계가 있는 엔티티를 조회하는 방법은 크게 2가지 이다.
1. 객체 그래프 탐색(객체 연관관계를 사용한 조회)
2. 객체지향 쿼리 사용(JPQL)

```java
// 객체 그래프 탐색
Member member = em.find(Member.class "member1");
Team team = member.getTeam();
```
```java
// 객체지향 쿼리 사용
private static void queryLogicJoin(EntityManager em) {
    String jpql = "select m from Member m join m.team t where" + "t.name=:teamName";

    List<Member> resultList = em.createQuery(jpql, Member.class).setParameter("teamName","팀1").getResultList();

    for(Member member : resultList){
        System.out.println("[query] memeber.userName="+member.getUsername());
    }
}
// 결과
[query] member.username=회원1
[query] member.username=회원2

/* 실행 과정
:teamName는 파라미터로 바인딩받는 문법이다.
select m from Member m join m.team t where t.name=:teamName

SELECT M.* FROM MEMBER MEMBER
INNER JOIN
    TEAM TEAM ON MEMBER.TEAM_ID = TEAM1_.ID
WHERE 
    TEAM1_.NAME="팀1"
*/
```

### 수정
```java
private static void update(EntityManager em){
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);

    Member member = em.find(Member.class, "member1");
    member.setTeam(team2);
}
// 수정은 em.update() 같은 메소드가 없다. 단순히 불러온 엔티티의 값만 변경해두면 트랜잭션을 커밋할 때 flush가 일어나면서 변경 감지 기능이 작동한다. 그리고 변경사항을 데이터베이스에 자동으로 반영한다. 이는 연관관계를 수정할 때도 같다.
```
### 연관관계 제거
```java
private static void deleteRelation(EntityManager em){
    Member member1 = em.find(Member.class, "member1");
    member1.setTeam(null);
}
```

### 연관된 엔티티 ㅅ학제
연관된 엔티티를 삭제하려면 **기존에 있던 연관관계를 먼저 제거하고 삭제해야 한다.**
```java
member1.setTeam(null);
member2.setTeam(null);
em.remove(team);
```

## 👐 양뱡향 연관관계
지금까지는 회원에서 팀으로 접근하는 다대일 단뱡향 매핑이였다. 이번에는 팀에서 회원으로 접근하는 관계를 추가해보자.
![image](https://user-images.githubusercontent.com/71022555/167242241-bf74de2c-062b-4428-8c1a-fee9a53edabc.png)  

### 양방향 연관관계 매핑
```java
// 회원 엔티티는 기존 그대로 진행

@Entity
public class Team{
    @Id
    @Coluimn(name ="TEAM_ID")
    private String id;
    
    private String name;

    // 추가된 부분
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
}
// 일대다 관계 이므로 팀 엔티티에 List<Member> members 리스트 컬렉션을 추가하였다. 
// mappedBy 속성은 양방향 매핑일 때 사용하는 반대쪽 매핑의 필드 이름을 갑으로 주면 된다. 반대쪽 매핑이 Team team이므로 team을 값으로 주었다.
```
### 일대다 컬렉션 조회
```java
public void biDirection(){
    Team team = en.find(Team.class "team1");
    List<Member> members = team.getMembers();

    for(Member member : members){
        System.out.println("member.username = "+member.getUsername());
    }
}
```
## 🕵️‍♂️ 연관관계의 주인
@OneToMany뿐만아니라 **mappedBy**는 왜 필요한 것인가?
> 객체에는 양방향 연관관계라는 것이 없기 때문이다.  
즉 **테이블은 키 하나로 두 테이블의 연관관계를 관리**한다.  
이 상황에서 엔티티를 양방향으로 매핑하면 회원 -> 팀, 팀 -> 회원 두 곳에서 서로를 참조나다. 따라서 이러한 상황에서 **참조는 둘 외래키는 하나**라는 문제가 발생한다.  
이런 차이로 인해 JPA에서는 **두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이것을 연관관계의 주인**이라 한다.  

### 연관관계 주인 규칙
- **연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)를 할 수 있다.** 반면에 주인이 아닌 쪽은 읽기만 할 수 있다.
- 주인은 **mappedBy**속성을 사용하지 않는다. 즉 mappedBy가 없는 쪽이 연관관계의 주인이다.
![image](https://user-images.githubusercontent.com/71022555/167242471-fdb3bf26-5f16-4e8a-a73f-ca79fdf6b6eb.png)  

```java
// 회원 -> 팀
class Member{
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}
// 팀 -> 회원
class Team{
    @OneToMany
    private List<Member> members = new ArrayList<Member>();
}
```
> **연관관계의 주인을 정한다는 것은 외래 키 관리자를 선택하는 것이다.**  
여기서는 회원 테이블에 있는 TEAM_ID 외래 키를 관리할 관리자로 선택하야 한다.  
여기서 회원 엔티티에 있는 Member.team을 주인으로 선택하면 자기 테이블에 있는 외래 키를 관리하면 된다.  
하지만 팀 엔티티에 있는 Team.members를 주인으로 선택하면 물리적으로 전혀 다른 테이블의 외래 키를 관리해야한다. 이 경우 Team.members가 있는 Team 엔티티는 TEAM 테이블에 매핑되어 있는데 관리해야할 외래 키는 MEMBER테이블에 있기 때문이다.
### 연관관게의 주인은 외래 키가 있는 곳
연관관계의 주인은 테이블에 외래 키가 있는 곳으로 정해야 한다.  
여기서는 회원 테이블의 외래키를 가지고 있으므로 Member.team이 주인이 된다.

## 양방향 연관관계 저장
```java
public void test(){
    // 팀1 저장
    Team team1 = new Team("team1", "1팀");
    em.persist(team1); // save()는 저장후 ID 값을 return한다. persist는 반환형이 void

    //회원 저장
    Memeber member1 = new Member("member1", "회원1");
    member1.setTeam(team1); // 연관관계 설정 memeber1 -> team1
    em.persist(member1);
}
// 양방향 연관관계는 연관관계의 주인이 외래 키를 관리한다. 따라서 주인이 아닌 방향은 값을 설정하지 않아도 DB에 외래 키 값이 정상 입력된다.
team1.getMembers{}.add(member1); // 무시 된다.(연관관계의 주인 X)

member1.setTeam(team1); //연관관계 설정(연관관계 주인)

```

## ❗ 양방향 연관관계의 주의점
연관관계 주인에는 값을 입력하지 않고 주인이 아닌 곳에만 값을 입력하는 것이다.

```java
// 주인이 아닌 곳에만 값 설정
public void testSave(){
    // 회원 1 저장
    Member member1 = new Member("member1", "회원1");
    em.persist(member1);

    // 회원 2 저장
    Member member2 = new Member("member2", "회원2");
    em.persist(member2);

    Team team1 = new Team("team1", "팀1");
    // 주인이 아닌 곳에만 연관관계 설정
    team1.getMembers{}.add(member1);
    team1.getMembers{}.add(member2);
    
    em.persist(team1)
}
// 결과 조회시
MEMBER_ID USERNAME TEAM_ID
member1   회원1     null
member2   회원2     null
```
외래 키 TEAM_ID에 team1이 아닌 null 값이 입력되어 있는데, 연관관계의 주인이 아닌 Team.members에만 값을 저장했기 때문이다. 즉 **연관관계의 주인만이 외래 키의 값을 변경할 수 있다.** 이 코드에서는 Member.team에 아무 값도 입력하지 않았다. 따라서 TEAM_ID 외래 키의 값도 null이 저장된다.

### 순수한 객체까지 고려한 양방향 연관관계
JPA를 사용하지 않는 순수한 객체 상태에서는 양쪽에 다 값을 입력해주어야 심각한 문제가 발생하지 않는다.
```java
public void testObject(){
    // 팀1
    Team team1 = new Team("team1", "팀1");
    Member member1 = new Member("member1", "회원1");
    Member member2 = new Member("member2", "회원2");    

    member1.setTeam(team1);
    member2.setTeam(team1);

    List<Member> members = team1.getMembers();

    System.out.println(members.size());    
        
}
// 결과는 0이 출력된다.
// Member.team에만 연관관계를 저장했다.
team1.getMembers{}.add(member1); // 처럼 양쪽 다 관계를 설정해야 한다.
```

### 둘 다 고려한 코드
```java
public void testPerfect(){

    // 팀1 저장
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    Member member1 = new Member("member1", "회원1");    

    // 양방향 연관관계 설정
    member1.setTeam(team1); // member1 -> team1
    team1.getMembers{}.add(member1);  // team1 -> member1
    em.persist(member1);

    Member member2 = new Member("member2", "회원2");  team1.getMembers{}.add(member2);
    em.persist(member2);  
}
```

### 리팩토링 한 코드
```java
public class Memeber{
    private Team team;
    
    public void setTeam(Team team){
        this.team = team;
        this.getMembers().add(this);
    }
}

Member.setTeam(team) // 메소드 하나로 양방향 관계를 모두 설정하도록 변경
```

### 편의 메소드 작성 시 유의 사항
```java
member1.setTeam(team1); 
member2.setTeam(team2); 
Member findMember = team1.getMember(); // member1이 여전히 조회된다.

// 기존에 team2로 변경할 때 team1 -> member1 관계를 제거하지 않아서 member1이 여전히 검색된다.

public void setTeam(Team team){
    if(this.team != null){
        this.team.getMembers().remove(this);
    }

    this.team = team;
    this.getMembers().add(this);
}
```