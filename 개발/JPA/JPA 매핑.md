## JPA 매핑

### 대표 매핑 어노테이션
- 객체와 테이블 매핑 : @Entity, @Table
- 기본 키 매핑 : @Id
- 필드와 컬럼 매핑 : @Column
- 연관관계 매핑 : @ManyToOne, @JoinColumn

## @Entity
- 기본 생성자는 필수
- final 클리스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안 된다.

|속성|기능|기본값|
|------|---|---|
|name|JPA에서 사용할 엔티티 이름을 지정한다. 보통 기본값이 클래스 이름을 사용한다.|설정하지 않으면 클래스 이름을 그대로 사용한다(예: Member)|

## @Table
- 엔티티와 매핑할 테이블을 지정한다.
- 생략시 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

|속성|기능|기본값|
|------|---|---|
|name|매핑할 테이블 이름|엔티티 이름을 사용한다.|
|catalog|catalog 기능이 있는 데이터베이스에 catalog를 매핑한다.||
|schema|schema 기능이 있는 데이터베이스에서 schema를 매핑한다.|DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 스키가 자동 생성 기능을 사용해서 DDL을 만들 때만 사용|

## 데이터베이스 스키마 자동 생성
```java
<porperty name="hiberante.hbm2ddl.auto" value="create"/>
// 애플리케이션 실행 시점에 데이터베이스 테이블을 자동으로 생성한다.
// hibernate.show_sql 속성을 true로 설정하면 콘솔에 실행되는 테이블 생성 DDL을 출력할 수 있다.
```

hibernate.hbm2ddl.auto 속성
|옵션|설명|
|------|---|
|create|기존 테이블을 삭제하고 새로 생성한다. DROP+CREATE|
|create-drop|create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL을 제거한다 DROP+CREATE+DROP|
|update|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정한다.|
|validate|데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션 실행 X 이 설정은 DDL 수정 X|
|none|자동 생성 기능을 사용하지 않으려면 속성 자체를 삭제하거나 유효하지 않은 옵션 값을 주면 된다 ex)none |
  
## DDL 생성 기능
- @Column(name="Name", nullable = false, length = 10)
    - not null 제약조건 추가 및 문자의 크기 10으로 제한
- @Table(name="Member", uniqueConstraints ={@UniqueConstraint{
    name = "NAME_AGE_UNIQUE",
    columnNames = {"NAME", "AGE"}
}})  

= ALTER TABLE MEMBER ADD CONSTARINT NAME_AGE_UNIQUE UNIQUE(NAME, AGE)

## 기본 키 매핑
-  ### 직접 할당 : 기본 키를 애플리케이션에 직접 한달한다.
- ### 자동생성 : 대리 키 사용 방식
    - IDENTITY : 기본키 생성을 데이터베이스에 위임한다.
    - SEQUENCE : 데이터베이스 시퀀스를 사용해서 기본 키를 할당 한다.
    - TABLE : 키 생성 테이블을 사용한다.

## 직접 할당 전략
@Id 로 매핑
- em.persist()로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접할당 하는 방법
```java
Test test = new Test();
test.setId("id1") // 기본 키 직접할 할당
em.persist(test);
```
  
## INDENTITY 전략
기본 키 생성을 데이터 베이스에 위임하는 전략으로 MYSQL, DB2, SQL SERVER 에서 사용한다.
- AUTO_INCREMENT 사용처럼 데이터베이스에 값을 저장하고 나서야 기본 키값을 구할 수 있을 떄 사용한다.
```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private int id;

Test test = new Test();
em.persist(test); // 처음 들어가니까 test.id = 1
// 저장 시점에 DB가 생성한 값을 JPA가 조회한다.
```
  
## SEQUENCE 전략
데이터베이스 시퀀스 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다. SEQUENCE 전략은 이 시퀀스를 사용해서 기본키를 사용한다. 시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용할 수 있다.
```java
@Entity
@SequenceGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "BOARD_SEQ", // 매핑할 DB 시퀀스 이름
    initialValue = 1, allocationSize =1)
    public class Board{

        @id 
        @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BOARD_SEQ_GENERATOR")
    }

```
1. @SequenceGenerator를 사용해서 BOARD_SEQ_GENERATOR라는 시퀀스 생성기를 등록
2. sequenceName 속성의 이름으로 BOARD_SEQ를 시정 JPA는 이 시퀀스 생성기를 실제 DB의 BOARD_SEQ와 매핑한다.
3. 키 생성 전략 SEQUENCE 등록
```java
Test test = new Test();
em.persist(test);
```
1. em.persist()를 호출할 때 먼저 DB 시퀀스를 사용해서 식별자를 조회한다.
2. 조회한 식별자를 엔티티에 할당.
3. 엔티티를 영속성 컨텍스트에 저장
  

### @SequenceGenerator
|속성|기능|기본값|
|------|---|---|
|name|식별자 생성기 이름|필수|
|sequenceName|DB에 등록되어 있는 시퀀스 이름|hibernate_sequence|
|initialValue|DDL 생성 시에만 사용됨. 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정한다.|1|
|allocationSize|시퀀스 한 번 호출에 증가하는 수|50|
|catalog, schema|데이터베이스 catalog, schema 이름||
  

## TABLE 전략
키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스를 흉내내는 전략. 모든 DB에 적용 가능
```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    sequenceName = "TABLE_SEQUENCES", // 매핑할 DB 테이블 이름
    pkColumnValue = "BOARD_SEQ", allocationSize =1)
    public class Board{

        @id 
        @GeneratedValue(strategy = GenerationType.TABLE, generator = "BOARD_SEQ_GENERATOR")
    }
```

### @TableGenerator
|속성|기능|기본값|
|------|---|---|
|name|식별자 생성기 이름|필수|
|table|키생성 테이블명|hibernate_sequences|
|pkColumnName|시퀀스 컬럼명|sequence_name|
|valueColumnName|시퀀스 값 컬럼명|next_val|
|pkColumnValue|키로 사용할 값 이름|엔티티 이름|
|initialValue|초기 값, 마지막으로 생성된 값이 기준이다.|0|
|allocationSize|시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)|50|
|catalog, schema|DB catalog, schema 이름||
|uniqueConstraints(DDL)|유니크 제약 조건을 지정할 수 있다.||
  

## AUTO 전략
GenerationType.AUTO는 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다. ex) Oracle -> SEQUENCE. MYSQL -> IDENTITY를 사용한다.  
@GeneratedValue의 기본값을 AUTO이다.
  

## 필드와 컬럼 매핑: 레퍼런스
|분류|매핑 어노테이션|설명|
|------|---|---|
|필드와 컬럼 매핑|@Column|컬럼을 매핑한다.|
|~|@Column|컬럼을 매핑한다.|
|~|@Enumerated|자바의 enum 타입을 매핑한다.|
|~|@Temporal|날짜 타입을 매핑한다.|
|~|@Lob|BLOB, CLOB 타입을 매핑한다.|
|~|@Transient|특정 필드를 데이터베이스에 매핑하지 않는다.|
|기타|@Access|JPA가 엔티티에 접근하는 방식을 지정한다|
  

### @Column
@Column은 객체 필드를 테이블 컬럼에 매핑한다. 가장 많이 사용되고 기능도 많다.
Column 속성
|속성|기능|기본값|
|------|---|---|
|name|필드와 매핑할 테이블의 컬럼 이름|객체의 필드 이름|
|insertable(거의 사용하지 않음)|엔티티 저장 시 이 필드도 같이 지정한다. false 설정 시 이 필드는 DB에 저장하지 않는다. false 옵션은 읽기 전용일 때 사용|true|
|updatable(거의 사용하지 않음)|엔티티 수정 시 이 필드도 같이 수정한다. false로 설정하면 DB에 수정히지 않는다 false 옵션은 읽기 전용일 때 사용|true|
|table(거의 사용하지 않음)|하나의 엔티티를 두 개 이상의 테이블에 매핑할 때 사용한다. 지정한 필드를 다른 테이블에 매핑할 수 있다.|현재 클래스에 매핑된 테이플|
|nullable(DDL)|null 값의 허용 여부를 설정한다. flase로 설정 시 DDL 생성 시 not null 제약조건이 붙는다.|true|
|unique(DDL)|@Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다. 만약 두 컬럼 이상을 사용해서 유니크 제약조건을 사용하려면 클래스 레벨에서 @Table.uniqueConstraints를 사용해야 한다.||
||데이터베이스 컬럼 정보를 직접 줄 수 있다|필드의 자바 타입과 방언 정보를 사용해서 절절한 컬럼 타입을 생성한다.|
|length(DDL)|문자 길이 제약조건, String 타입에만 사용한다.|255|
|precision,scale(DDL)|BigDecimal(BigInteger) 타입에서 사용한다. precision은 소수점을 포함한전체자릿수, scale은 소수의 자릿수다.|precision=19, scale=2|
  

### @Enumerated
자바의 enum 타입을 매핑할 때 사용한다.
|속성|기능|기본값|
|------|---|---|
|value|- EnumType.ORDINAL: enum 순서를 DB에 저장 - EnumType.STRING: enum 이름을 DB에 저장|EnumType.ORDINAL| 
사용예시
```java
// enum 클래스
enum RoleType{
    ADMIN, USER
}

// 매핑
@Enumerated(EnumType.STRING)
private RoleType roleType;

// 사용
member.setRoleType(RoleType.ADMIN); // -> DB에 문자 ADMIN으로 저장된다.
```
- EnumType.ORDINAL은 enum에 정의된 순서대로 ADMIN은 0, USER는 1 값이 DB에 저장된다.
    - 장점 : DB에 저장되는 데이터의 크기가 작다.
    - 단점 : 이미 저장된 enum의 순서를 변경할 수 없다.
- EnumType.STRING은 enum 이름 그대로 ADMIN은 'ADMIN' USER는 'USER"라는 문자로 DB에 저장된다.
    - 장점 : 저장된 enum의 순서가 바뀌거나 enum이 추가되어도 안전하다.
    - 단점 : DB에 저장되는 데이터 크기가 ORDINAL에 비해서 크다.
  

### @Temporal
날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용한다.
|속성|기능|기본값|
|------|---|---|
|value|-TemporalType.DATE: 날짜, DB date 타입과 매핑 - TemporalType.TIME: 시간, DB time 타입과 매핑 - TemporalType.TIMESTAMP: 날짜와 시간, DB timestamp 타입과 매핑|TemporalType은 필수로 지정해야 한다.|
사용예시
```java
@Temporal(TemporalType.DATE)
private Date date; // 날짜

@Temporal(TemporalType.TIME)
private Date time;  // 시간

@Temporal(TemporalType.TIMESTAMP)
private Date timestamp; // 날짜와 시간

// @Temporal 을 생략하면 Date와 가장 유사한 timestamp로 정의된다. MySQL은 datetime으로 자동 변환
// 생성된 DDL //
date date,
time time,
timestamp timestamp,
```
  

### @Lob
데이터베이스 BOLB, CLOB 타입과 매핑한다.
@Lob에는 지정할 수 있는 속성이 없다. 대신 매핑하는 필드 타입이 문자면 CLOB으로 매핑하고 나머지는 BLOB으로 매핑한다.
- CLOB : String, char[], java.jql.CLOB
- BLOB : byte[], java.jsql.BLOB
사용예시
```java
@Lob
private String lobString;

@Lob
private byte[] lobByte;

// 생성된 DDL
// 오라클
lobString clob,
lobByte blob,

//MYSQL
lobString longtext,
lobByte longblob,

//PostgreSQL
lobString text,
lobByte old
```
  

### @Transient
이 필드는 매핑하지 않는다. 따라서 DB에 저장히자 않고 조회하지도 않는다. 객체에 임시로 어떤 값을 보관하고 싶을 떄 사용
```java
@Transient
private Integer tmp;
```
  

### @Access
JPA가 엔티티 데이터에  접근하는 방식을 지정한다.
- 필드 접근 : AccessType.FIELD로 지정한다. 필드에 직접 접근한다. 필드 접근 권환이 private이어도 접근 가능
- 프로퍼티 접근 : AccessType.PROPERTY로 지정한다. 접근자를 사용한다.
@Access를 설정하지 않으면 @Id의 위치릴 기준으로 접근 방식이 설정된다.
```java
// 필드 접근 코드
@Entity
@Access(AccessType.FIELD)

public class Test{
    @Id
    private String id;
}

@Entity
@Access(AccessType.PROPERTY)

// 프로퍼티 접근 코드
public class Test{
    
    private String id;

    @Id // @Id가 프로퍼티에 있으므로 @Access(AccessType.PROPERTY)로 설정한 것과 같다. 따라서 @Access 생략가능
    public String getId(){
        return id;
    }
}

// 둘다 동시 사용
@Entity

public class Test{
    @Id
    private String id;

    @Transient
    private String firstName;
    @Transient
    private String lastName;

    @Access(AccessType.PROPERTY)
    public String getFullName(){
        return firstName + lastName;
    }
    // @id가 필드에 있으므로 기본은 필드 접근 방식을 사용하고 getFullName()만 프로퍼티 접근 방식을 사용한다. 따라서 회원 엔티티를 저장하면 회원 테이블의 FULLNAME 컬럼에 firstName + lastName의 결과가 저장됩니다.
}
```
