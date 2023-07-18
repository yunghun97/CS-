# 😮 JPA N+1 문제

### 연관 관계에서 발생하는 이슈로 엔티티를 조회할 경우에 조회된 데이터 갯수(n) 만큼 연관관계의 조회 쿼리가 추가로 발생하여 데이터를 읽어오게 된다. 이를 N+1 문제라고 한다.

## 연관관계  
![image](https://user-images.githubusercontent.com/71022555/172904624-1be430ec-28c5-4d8b-a5e5-3bb129e77a9c.png)  
  
```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 10, nullable = false)
    private String name;

    @OneToMany(mappedBy = "user")
    private Set<Article> articles = emptySet();
}

@Entity
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 50, nullable = false)
    private String title;

    @Lob
    private String content;

    @ManyToOne
    private User user;
}
// FetchType 기본 값 : ~ToMany(fetch = FetchType.EAGER)
// FetchType 기본 값 : ~ToOne(fetch = FetchType.LAZY)
```

## 😅 즉시로딩(EAGER)  
```java
userRepository.findById("Gwon");
```
- 내부적으로 inner join문 하나가 날아가서 User가 조회됨과 동시에 Article까지 즉시로딩되는 것을 확인할 수 있음
- 문제는 jpql에서 발생한다.
    - select u from User u; 구문에서 FetchType는 EAGER로 설정되어 있으므로 select한 모든 User에 대해서 article가 있는지 검색을 진행하게 된다.
- 따라서 자원의 낭비가 발생한다.

## 😀 지연로딩(LAZY)
```java
List<User> users = userRepository.findAll();
for(User user : users){ // N+1 문제 발생 
    System.out.println(user.articles().size());
}
```
- 연관된 객체를 "사용"하는 시점에 로딩을 해주는 방법
- 지연로딩을 설정하더라도 N+1은 발생할 수 있다!
    - 해당 연결 entity에 대해 프록시를 걸어두고, 사용할 때 쿼리문을 결국 날리기 때문에 처음 find할 때는 N+1이 발생하지 않지만 추가로 user 검색 후 User의 Article을 사용해야 한다면 이미 캐싱된 User의 Article 프록시에 대한 쿼리가 또 발생하게 됩니다.
- 지연로딩을 했으니 지연로딩 대상인 Article을 나중에 조회해서 쿼리가 또 날아가는 것조차도 N+1 문제의 예 중 하나

## ✨ 지연로딩에서의 해결책 - fetch join
### N+1 이 발생하는 원인
> JPA가 자동으로 먼저 생성해주는 Jpql을 통해서 우선적으로 쿼리를 만들다보니 연관관계가 걸려있어도 join이 바로 걸리지 않는다.
```java
// N+1 문제 발생 코드 (프록시로 가져오는건 변함이 없음)
@Query("select distinct u from User u left join u.articles")
List<User> findAllJPQL();
```

## fecth join 방법 1
```java
@Query("select distinct u from User u left join fetch u.articles")
List<User> findAllJPQLFetch();
```

```java
List<User> users = userRepository.findAll();
for(User user : users){ // N+1 문제 발생 
    System.out.println(user.articles().size());
}
// 쿼리를 날릴 때 article을 한번에 가져온다.
```
  
## fetch join 방법 2 **@EntityGraph**
- 기존의 Jpql fetch join 사용 시 하드코딩을 하게 된다는 단점이 있습니다.
```java
@EntityGraph(attributePaths = {"articles"}, type = EntityGraphType.FETCH)
@Query("select distinct u from User u left join u.articles")
List<User> findAllEntityGraph();
// Jpql 구문에서의 fetch는 존재하지 않지만 기존과 마찬가지로 fetch join을 통해 바로 조회할 수 있음
```

## ❗ fetch join 주의할 점
## 🚗 문제점 1. Pagination
- Page를 반환하는 쿼리를 작성 시 해당 오류가 발생
```java
@EntityGraph(attributePaths = {"articles"}, type = EntityGraphType.FETCH)
@Query("select distinct u from User u left join u.articles")
Page<User> findAllPage(Pageable pageable);
```
```java
void pagingFetchJoinTest() {
    PageRequest pageRequest = PageRequest.of(0, 2);
    Page<User> users = userRepository.findAllPage(pageRequest);
    for (User user : users) {
        System.out.println(user.articles().size());
    }
}
```
- 해당 쿼리를 자세히 보면 페이징 처리를 할 때 사용하는 Limit, Offset이 없습니다. 
- user.articleSize()은 정상적으로 작동
```bash
WARN 79170 --- [    Test worker] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
# applying in memory, 즉 인메모리를 적용해서 조인을 했다고 합니다.
```
> **즉 List의 모든 값을 select해서 인메모리에 저장하고 application 단에서 필요한 페이지만큼 반환을 진행해줌 -> Paging을 한 이유가 없어짐 + Out Of Memory 발생 확률 상승**
  
> 따라서 Pagination에서는 fetch join을 하고싶어서 한다고 하더라고 해결을 할 수 없습니다.

```
짧게 이야기를 하자면 fetch join에서 distinct를 쓰는 것과 연관이 있습니다. distinct를 쓰는 이유는 하나의 연관관계에 대해서 fetch join으로 가져온다고 했을 때 중복된 데이터가 많기 때문에 실제로 원하는 데이터의 양보다 중복되어 많이 들어오게 됩니다.

그 이유때문에 개발자가 직접 distinct를 통해서 jpa에게 중복 처리를 지시하게 되는 것이고, Paging처리는 쿼리를 날릴 때 진행되기 때문에 jpa에게 pagination 요청을 하여도 jpa는 distinct때와 마찬가지로 중복된 데이터가 있을 수 있으니 limit offset을 걸지 않고 일단 인메모리에 다 가져와서 application에서 처리하는 것이죠.
```
  
## 🍕 Pagination 해결책 1 : ~ToOne 관계에서 페이징 처리
- User에서 검색하지 않고 Atricle은 User에 대해서 @ManyToOne 연관관계이기 때문에 Pagination을 진행해도 인메모리에서 모든 Article을 조회하는 것이 아닌 limit을 걸어 필요한 데이터만 가져올 수 있습니다.
## 🍔 Pagination 해결책 2 : Batch Size
- 컬랙션 조인을 하는 경우에는 **fetch join을 아예 사용하지 않고 조회할 컬랙션 필드에 대해서 @BatchSize** 를 걸어 해결합니다.
```java
@BatchSize(size=100)
@OneToMany(mappedBy="user", fetch=FetchType.LAZY)
private Set<Article> articles = emptySet();

Page(User) findAll(Pageable pageable);
```
- article을 select하는 쿼리가 하나 등장합니다.
- 지연로딩하는 객체에 대해서 Batch성 loading을 하는 것이라고 생각하면 됩니다.,
- 기존의 지연로딩에 대해서는 객체를 조회할 때 그때그때 쿼리문을 날려서 N+1 문제가 발생한 반면 
- 객체를 조회하는 시점에 쿼리를 하나만 날리는게 아니라 해당하는 Article에 대해서 쿼리를 batch size개를 날리는 것입니다.
```sql
-- batch 쿼리에서 where부분
where
	articles0_.user_id in (
		?, ?
	)
```
- in(?, ?)가 결국 user id를 100개를 가져오는 쿼리문으로써 그때그때 조회하는 것이 아닌 조회할 때 그냥 **batch size**만큼 한번에 가져와서 뒤에 생길 지연로딩에 대해서 미연에 방지(batch size는 Article이 아닌  User 기준)
> 최적화된 데이터 사이즈를 알지 못한다면 적절하지 않을 수 있다.
  
## 🍟 Pagination 해결책 3 : @Fetch(FetchMode.SUBSELECT)
- @BatchSize와 비슷하지만 다른 어노테이션입니다.
- @BatchSize의 경우 사이즈 갯수 제한을 임의로 두어서 사용자가 최적화된 데이터 사이즈를 적용하게끔 도와준다면 이 어노테이션은 그냥 전부 진행
```java
@Fetch(FetchMode.SUBSELECT)
@OneToMany(mappedBy="user", fetch=FetchType.LAZY)
private Set<Article> articles = emptySet();
```
- @BathSize(size="INF") 느낌
- Batch Size의 경우 주어진 size만큼 User ID를 입력하여 프록시 상태에 따라 지연로딩을 진행했다면 지금은 User Id를 전부 조회한다.

## 🚓 문제점 2. 둘 이상의 Collection fetch join(~ToMany) 불가능

- fetch join을 할 때 ToMany의 경우 한번에 fetch join을 가져오기 때문에 collection join이 2개이상이 될 경우 너무 많은 값이 메모리로 들어와 exception(MultipleBagFetchException)이 추가로 걸립니다.
- 2개 이상의 bags, 즉 collection join이 두개이상일 때 exception이 발생합니다.
참고자료
- ~ToOne은 얼마만큼 fetch join을 해도 괜찮지만 ~ToMany는 하나일 때는 인메모리에서 처리하고 두 개이상은 Exception으로 제한
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 10, nullable = false)
    private String name;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Article> articles = new ArrayList<>();
    
    @OneToMany(mappedBy = "question", fetch = FetchType.LAZY)
    private List<Question> questions = new ArrayList<>();
}

@EntityGraph(attributePaths = {"articles", "questions"}, type = EntityGraphType.FETCH)
@Query("select distinct u from User u left join u.articles")
List<User> findAllEntityGraph2();

// org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags: [com.example.jpa.domain.User.articles, com.example.jpa.domain.User.questions]; nested exception is java.lang.IllegalArgumentException: org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags: [com.example.jpa.domain.User.articles, com.example.jpa.domain.User.questions] 에러 발생
```
### Collection fetch join 해결책 1. 자료형을 Set
- Set을 사용하게 된다면 HashSet으로는 순서가 중요한 데이터에는 순서를 보장할 수 없기 때문에 LinkedHashSet을 사용권장
``` 
다만 Set을 사용한다고 해서, Pagination은 마찬가지로 해결이 불가능합니다.

Pagination은 근본적으로 몇개의 collection join 있던 간에 인메모리에서 가져오기 때문에 OOM을 발생시킬 수 있는 원인이 되어 해당 방법으로는 해결이 불가능합니다.

설령 Collection join이 한 개인 상황에서 Set 자료구조를 사용한다고 해도 인메모리에서 가져옵니다.

HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```
### Collection fetch join 해결책 1. BatchSize
- Pagination의 해결책 이지만 Collection join이 두 개 이상일 때 MultipleBagFetchException을 해결할 수 있는 방법이기도 합니다.
```java
@BatchSize(size = 100)
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private Set<Article> articles = emptySet();

@Query("select distinct u from User u left join u.articles left join u.questions")
Page<User> findAllPage2(Pageable pageable);
```
> **주의해야할 점은 batch size에 fetch join을 걸면 안됩니다.**
```
fetch join이 우선시되어 적용되기 때문에 batch size가 무시되고 fetch join을 인메모리에서 먼저 진행하여 List가 MultipleBagFetchException가 발생하거나, Set을 사용한 경우에는 Pagination의 인메모리 로딩을 진행합니다.
```

## 💙 종합
- **즉시로딩**
    - jpql을 우선적으로 select하기 때문에 즉시로딩을 이후에 보고 또다른 쿼리가 날아가 N+1
- **지연로딩**
    - 지연로딩된 값을 select할 때 따로 쿼리가 날아가 N+1
- **fetch join**
    - 지연로딩의 해결책
    - 사용될 때 확정된 값을 한번에 join에서 select해서 가져옴
    - Pagination이나 2개 이상의 collection join에서 문제가 발생
- **Pagination**
    - fetch join 시 limit, offset을 통한 쿼리가 아닌 인메모리에 모두 가져와 application단에서 처리하여 OOM 발생
    - BatchSize를 통해 필요 시 배치쿼리로 원하는 만큼 쿼리를 날림 > 쿼리는 날아가지만 N번 만큼의 무수한 쿼리는 발생되지 않음
- **2개 이상의 Collection join**
    - List 자료구조의 2개 이상의 Collection join(~ToMany관계)에서 fetch join 할 경우 MultipleBagFetchException 예외 발생
    - Set자료구조를 사용한다면 해결가능 (Pagination은 여전히 발생)
    - BatchSize를 사용한다면 해결가능 (Pagination 해결)



참고자료
---
#### https://incheol-jung.gitbook.io/docs/q-and-a/spring/n+1
#### https://velog.io/@jinyoungchoi95/JPA-%EB%AA%A8%EB%93%A0-N1-%EB%B0%9C%EC%83%9D-%EC%BC%80%EC%9D%B4%EC%8A%A4%EA%B3%BC-%ED%95%B4%EA%B2%B0%EC%B1%85