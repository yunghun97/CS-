# 🤠 RequiredArgsConstructor (생성자 주입)

## @RequiredArgsConstructor
이 어노테이션은 초기화 되지않은 **final**필드나, **@NonNull**이 붙은 필드에 대해 생성자를 생성해 줍니다. 주로 의존성 주입(Dependency Injection) 편의성을 위해서 사용한다.

## 의존성 주입 종류
1. Constructor(생성자)
```java
public class Example{
    private final AService aService;
    @Autowired
    public Example(AService aService){
        this.aService = aService;
    }
}
```
2. Setter
```java
public class Example{
    private AService aService;
    private BService bService;

    @Autowired
    public void setAService(AService aService){
        this.aService = aService;
    }
    @Autowired
    public void setBService(BService bService){
        this.bService = bService;
    }
}
```
3. Field
```java
public class Example{
    @Autowired
    private AService aService;
}
```

## @RequiredArgsConstructor 어노테이션을 사용한 생성자 주입
생성자의 단점은 위의 생성자 코드를 만들기 번거롭다. 이를 lombok을 사용하여 간단한 방법으로 생성자 주입 방식의 코딩 가능
### @RequiredArgsConstructor
**final**이 붙거나 **@NotNull**이 붙은 필드의 생성자를 자동 생성해주는 lombok 어노테이션
1. 기존의 filed 주입 방식
```java
@Service
public class Test{
    @Autowired
    private AService aService;
}
```
2. @RequiredArgsConstructor 를 활용한 생성자 주입
```java
@Service
@RequiredArgsConstructor
public class Test{
    private final AService aService;
}
```
3. @RequiredArgsConstructor 를 사용하지 않을 시
```java
public class Example{
    private final AService aService;
    @Autowired
    public Example(AService aService){
        this.aService = aService;
    }
}
```