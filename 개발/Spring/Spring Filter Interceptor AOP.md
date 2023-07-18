# 😀 Spring Filter Interceptor AOP

### 자바 웹 개발을 하다보면, **공통적**으로 처리해야 할 업무들이 많다.

> 즉 공통 부분들을 빼서 따로 관리하는 것이 효율적이다.
  
### 공통 처리 방법 
1. Filter
2. Interceptor
3. AOP  
  

## 흐름
![image](https://user-images.githubusercontent.com/71022555/172141138-28323e4d-5f95-480b-9ad3-3fc28decefbd.png)  
  
- Interceptor와 Filter는 Servlet 단위에서 실행된다.
- AOP는 메소드 앞에 Proxy 패턴의 형태로 실행된다.
- 실행 순서 Filter -> Interceptor -> AOP -> Interceptor -> Filter
  
1. 서버를 실행시켜 서블릿이 올라오는 동안에 init이 실행되고 그 후 doFilter가 실행된다.
2. 컨트롤러에 들어가기 전 preHandler가 실행된다.
3. 컨트롤러에서 나와 postHandler, after Completion, doFilter 순으로 진행이된다.
4. 서블릿 종료 시 destory가 실행된다.
  
## 😮 개념
### 1. Filter(필터)
- 요청과 응답을 거른뒤 정제하는 역할을 한다.
- 서블릿 필터는 DispatcherServlet 이전에 실행이 되는데 필터가 동작하도록 지정된 자원의 앞단에서 요청내용을 변경하거나,  여러가지 체크를 수행할 수 있다.
- 자원의 처리가 끝난 후 응답내용에 대해서도 변경하는 처리를 할 수가 있다.
- 보통 web.xml에 등록하고, 일반적으로 인코딩 변환 처리, XSS방어 등의 요청에 대한 처리로 사용된다.  
### 실행 메서드
1. init() - 필터 인스턴스 초기화  
2. doFilter() - 전/후 처리  
3. destroy() - 필터 인스턴스 종료  
  
### 2. 인터셉터(Interceptor)
- 요청에 대한 작업 전/후로 가로챈다고 보면 된다.
- 필터는 스프링 컨텍스트 외부에 존재하여 스프링과 무관한 자원에 대해 동작한다. 
- 하지만 인터셉터는 스프링의 DistpatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 스프링 컨텍스트(Context, 영역) 내부에서 Controller(Handler)에 관한 요청과 응답에 대해 처리한다.
- **스프링의 모든 빈 객체에 접근할 수 있다.**
- 여러개 사용 가능
- 로그인 체크, 권한 체크, 프로그램 실행시간 계산작업, 로그확인 등
### 실행 메서드
1. preHandler() - 컨트롤러 메서드가 실행되기 전
2. postHanler() - 컨트롤러 메서드 실행직 후 view페이지 렌더링 되기 전
3. afterCompletion() - view페이지가 렌더링 되고 난 후
  
### 3. AOP
OOP를 보완하기 위해 나온 개념
- 객체 지향의 프로그래밍을 했을 때 중복을 줄일 수 없는 부분을 줄이기 위해 종단면(관점)에서 바라보고 처리한다.
- 로깅, 트랜잭션, 에러 처리 등 비즈니스단의 메서드에서 조금 더 세밀하게 조정하고 싶을 때 사용합니다.
- Interceptor나 Filter와는 달리 메소드 전후의 지점에 자유롭게 설정이 가능하다.
- **Interceptor와 Filter는 주소로 대상을 구분해서 걸러내야하는 반면, AOP는 주소, 파라미터, 어노테이션 등 다양한 방법으로 대상 지정 가능**  
- AOP의 Advice와 HandlerInterceptor의 가장 큰 차이는 파라미터의 차이다.
    - Advice의 경우 JoinPoint나 ProceedingJoinPoint 등을 활용해서 호출한다.
    - 반면 HandlerInterceptor는 Filter와 유사하게 HttpServletRequest, HttpServletResponse를 파라미터로 사용한다.

### AOP 용어
|이름|설명|
|:---|:---|
|JoinPoint|공통 기능이 삽입되어 동작될 수 있는 위치|
|PointCut|조인포인트 중에서 실행 하려는 위치를 결정|
|Advice|실제로 적용하는 기능(로그, 트랜잭션, 인증)으로 적용하는 시점을 가지고 있다.|
|Target|공통기능이 적용될 대상 클래스 객체|
|Weaving|Advice를 비즈니스 로직 코드에 삽입한느 것|
|Aspect|여러클래스나 기능에 사용되는 관심사를 모듈화함(포인트 컷 + 어드바이스) ex) 트랜잭션 관리 기능|
  
### Advice 종류
|이름|설명|
|:---|:---|
|@Before|미리 정의한 Pointcut 진입전에 수행|
|@AfterReturning|미리 정의한 Pointcut에서 return이 발생한 후에 수행|
|@AfterThrowing|미리 정의한 Pointcut에서 예외가 발생할 경우 수행|
|@After|미리 정의한 Pointcut에서 예외 발생 여부와 상관없이 무조건 수행|
|@Around|미리 정의한 Pointcut의 전,후에 필요한 동작을 수행|
  
### 명시자 execution
- execution(AspectJ 표현식)
> execution(제한자패턴? 리턴타입패턴? 이름패턴(파라미터패턴)) ? - 생략가능  

### AspectJ 표현식
|와일드카드|연산자|
|:---|:---|
|*|1개의 모든 값을 표현, 매개변수 사용(1개의 파라미터), 패키지 사용(1개의 패키지)|
|..|0개 이상의 값을 표현|
|+|주어진 타입의 자식 클래스나 인터페이스를 표현|  
  
### AspectJ 표현식 적용 예시
|사용|설명|
|:---|:---|
|execution(public * *(..)) |public 메소드 선택|
|execution(* set*(..))|이름이 set으로 시작하는 모든 메소드|
|execution(* get*(..)) |이름이 get으로 시작하는 모든 메소드|
|execution(* main(..))|이름이 main인 메소드|
|execution(* com.ssafy..(..))|com.ssafy 패키지에 있는 클래스의 모든 메서드|

### API 클래스
- JoinPoint
    - Object getTarget() 핵심 기능 클래스 객체
    - Signature getSignature() 핵십기능 클래스의 메서드
- ProceedingJointPoint
    - Signature getSignature() 핵심기능 클래스의 메서드
    - proceed() 핵심기능 클래스의 메서드 실행
  
### AOP 적용
- 클래스 위쪽에 @Aspect 선언
- 메서드 위쪽에 advice 선언
  

#### 참고자료
---
##### https://goddaehee.tistory.com/154