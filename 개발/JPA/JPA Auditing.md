# 😗 JPA Auditing
  
### Java에서 ORM 기술인 JPA를 사용하여 도메인을 관계형 데이터베이스 테이블에 매핑할 때 공통적으로 도메인들이 가지고 있는 필드나 컬럼들이 존재합니다. ex) 생성, 수정일자, 식별자
### 이를 JPA에서 Audit(감시하다, 감사하다)라는 뜻으로 Spring Data JPA에서 시간에 대해서 자동으로 값을 넣어주는 기능입니다.  
### 도메인을 영속성 컨텍스트에 저장하거나 조회를 수행한 후에 update를 하는 경우 매번 시간 데이터를 입력해주어야 하는데 이를 audit을 이용하여 자동으로 시간을 매핑하여 데이터베이스의 테이블에 넣어주게 됩니다.

> Java 1.8 이상부터는 LocalDate, LocalDatTime을 사용합니다. 또한 LocalDateTime 객체 테이블 매핑 이슈는 Hibernate 5.2부터 해결되었습니다.  

  
## 😉 사용법
- JPA 의존성 추가
1. 어노테이션 추가(@EnableJpaAuditing)
```java
@EnableJpaAuditing
public class Application{
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```  
2. 공통으로 사용할 엔티티 추가
```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseDateEntity {
    @CreatedDate
    private LocalDate createdDate;
    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```
## 😶 속성
|속성|설명|
|:---|:---|
|@MappedSuperclass|JPA Entity 클래스들이 해당 추상 클래스를 상속할 경우 createDate, modifiedDate를 컬럼으로 인식|
|@EntityListeners(AuditingEntityListener.class|해당 클래스에 Auditing 기능을 포함|
|@CreatedDate|Entity가 생성되어 저장될 때 시간이 자동 저장|
|@LastModifiedDate|조회한 Entity의 값을 변경할 때 시간이 자동 저장|

참고자료 
---
#### https://webcoding-start.tistory.com/53