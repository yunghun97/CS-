# 📊 Spring Async

## @Async
Async Annotation은 Spring에서 제공하는 Thread Pool을 활용하는 비동기 메소드 지원 Annotation이다.

## 기존의 Java에서의 비동기 방식으로 메소드를 구현할 때
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class JavaAsync {

    static ExecutorService executorService = Executors.newFixedThreadPool(5);

    public void asyncMethod(final String message) throws Exception {
        executorService.submit(new Runnable() { // Runnable
            @Override
            public void run() {
                // do something
            }            
        });
    }
}
```
**java.util.concurrent.ExecutorService**을 활용해서 비동기 방식의 method를 정의 할 때마다 위와 같이 **Runnable**의 **run()**을 재구현해야 하는 등 동일한 작업들의 반복이 잦았다.

## @Async 사용
**@Async** Annotation을 활용하면 손쉽게 비동기 메소드 작성이 가능하다.  
만약 **Spring Boot**에서 간단히 사용 방법
1. **Application** Class에 **@EnableAsync** Annotation을 추가하고
```java
@EnableAsync
@SpringBootApplication
public class SpringBootApplication {
    ...
}
```
2. 비동기로 작동하길 원하는 method위에 **@Async** Annotation을 붙여주면 사용할 수 있다.
```java
public class AsyncTest{
    @Async
    public void asyncMethod() 
}
```


### 본인 개발 환경에 맞게 Customize하기 위해서는 **AsyncConfigurerSupport**를 상속받는 Class를 작성하는 것이 좋다.

## AsyncConfigurerSupport
```java
import java.util.concurrent.Executor;

import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurerSupport;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(30);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("DDAJA-ASYNC-");
        executor.initialize();
        return executor;
    }
}
// 위와 같이 설정한다면 최초 5개의 스레드에서 처리하다가 처리 속도가 밀릴 경우 100개 사이즈 queue에서 대기하고 그 보다 많은 요청이 발생할 경우 최대 30개 스레드까지 생성해서 처리하게 됩니다.
```
### 설정 요소
- **@Configuration** : Spring 설정 관련 Class로 @Component 등록되어 Scanning 될 수 있다.
- **@EnableAsync** : Spring method에서 비동기 기능을 사용가능하게 활성화 한다.
- **CorePoolSize** : 기본 실행 대기하는 Thread의 수**
- **MaxPoolSize** : 동시 동작하는 최대 Thread의 수
- **QueueCapacity** : MaxPoolSize 초과 요청에서 Thread 생성 요청시,
해당 요청을 Queue에 저장하는데 이때 최대 수용 가능한 Queue의 수,
Queue에 저장되어있다가 Thread에 자리가 생기면 하나씩 빠져나가 동작
- **ThreadNamePrefix** : 생성되는 Thread 접두사 지정

  
## 주의 사항
1. private method는 사용 불가, public method만 사용 가능
2. self-invocation(자가 호출) 불가, 즉 innermethod는 사용 불가
3. QueueCapacity 초과 요청에 대한 비동기 method 호출 시 방어 코드 작성

참고자료  

## Async 동작 방식
### AOP가 적용되어 Spring Context에서 등록된 Bean Object의 method가 호출 될 시에, Spring이 확인할 수 있고 @Async가 적용된 method의 경우 Spring이 method를 가로채 다른 Thread에서 실행 시켜주는 동작 방식이다.  
### 이 때문에 Spring이 해당 @Async method를 가로챈 후, 다른 Class에서 호출이 가능해야 하므로, private method는 사용할 수 없는 것이다.  
### 또한 Spring Context에 등록된 Bean의 method의 호출이어야 Proxy 적용이 가능하므로, inner method의 호출은 Proxy 영향을 받지 않기에 self-invocation이 불가능하다.

## QueueCapacity 초과 요청 방어 코드 작성
이번엔 **AsyncConfig**에서 **PoolSize**와 **QueueCapacity**를 줄여보고 위 코드를 다시 실행해보자.
```java
@Service
public class TestService {
    @Async
    public void asyncMethod(int i) {
        try {
            Thread.sleep(500);
            log.info("[AsyncMethod]"+"-"+i);
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
}
@AllArgsConstructor
@Controller
public Class TestController {

    private TestService testService;

    @GetMapping("async")
    public String testAsync() {
        log.info("TEST ASYNC");
        for(int i=0; i<50; i++) {
            testService.asyncMethod(i);
        }
        return "";
    } 
}
@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(10);
        executor.setThreadNamePrefix("DDAJA-ASYNC-");
        executor.initialize();
        return executor;
    }
}
// queued 된 tasks = 10개로 최대 수용 가능한 Thread Pool 수와 QueueCapacity 까지 초과된 요청이 들어오자,
// Task를 Reject하는 Exception이 발생하였다.
// i = 0 ~49 즉 기존에는 50개의 QueueSize를 사용하여 진행해도 오류가 없었지만 현재는 10개의 QueueSize를 사용하고 MaxPoolSize 10으로 인해 작업이 밀려 MaxPoolSize와 setQueueCapacity까지 초과된 요청이 들어와서
// Task를 Reject를 하는 Exception이 발생하였다.
```

## TaskRejectedException Handling
```java
@AllArgsConstructor
@Controller
public Class TestController {

    private TestService testService;

    @GetMapping("async")
    public String testAsync() {
        log.info("TEST ASYNC");
        try {
            for(int i=0; i<50; i++) {
                testService.asyncMethod(i);
        } catch (TaskRejectedException e) {
            // ....
        }
        return "";
    }
    
}
```
---
[gillog](https://velog.io/@gillog/Spring-Async-Annotation%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%A9%94%EC%86%8C%EB%93%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
[dzone](https://dzone.com/articles/effective-advice-on-spring-async-part-1)