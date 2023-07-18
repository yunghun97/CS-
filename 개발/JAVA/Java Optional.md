# 😃 Java Optional

### Java 8 부터 지원하는 클래스

## 😮 NPE(NullPointerException)
- NPE를 피하기위해서는 null 검사 로직을 추가해줘야 하는데 이를 많은 null 검사를 진행하면 코드가 복잡해지고 번거랍고 된다.

```java
List<Integer> list = getList();
// getList에서의 return 값이 null이라면 NPE가 발생된다.
System.out.print(list.size());

if(list!=null){ // null 체크
    System.out.print(list.size());
}
```

## 😀 Optional
- Java 8 에서는 Optional<T> 클래스를 사용해 NPE를 방지할 수 있도록 도와준다. 
- Optional<T>는 null이 올 수 있는 값을 감싸는 Wrapper 클래스로, 참조하더라도 NPE가 발생하지 않도록 도와준다. 
- Optional 클래스는 아래와 같은 value에 값을 저장하기 때문에 값이 null이더라도 바로 NPE가 발생하지 않으며, 클래스이기 때문에 각종 메소드를 제공해준다.

```java
public final class Optional<T>{
    private final T value;
    ...
}
```

## 🍕 Optional 생성하기
> value는 String 변수라 가정
1. 무조건 null이 아닌 값을 Optional 생성할 때
```java
Optional<String> optional = Optional.of(value); 
// value 변수의 값이 null인 경우에 NPE 발생
```
2. null이여도 상관이 없는 경우
```java
Optional<String> optional = Optional.ofNullable(value);
// value 변수의 값이 null일 수 있으며 null일 경우 Optional.empty()가 return 된다.
```
3. 빈 Optional 객체 생성
```java
Optional<String> optional = Optional.empty();
// Optional 객체 자체는 있지만 내부에서 가리키는 참조가 없는 경우이며 Optional.empty() 객체는 미리 생성되어 있는 싱글턴 인스턴스다.
```
```java
public final class Optional<T> {
    /**
     * Common instance for {@code empty()}.
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;
    private Optional() {
        this.value = null;
    }
}
```
- static 변수로 EMPTY 객체를 미리 생성해서 가지고 있다. 이러한 이유로 빈 객체를 여러 번 생성해줘야 하는 경우에도 1개의 EMPTY 객체를 공유함으로써 메모리를 절약하고 있다.

## 🎃 Optional 사용법 - 중간 처리
- Optional 객체를 가져와서 중간에 처리하고 다시 Optional 객체를 반환하는 메소드들이 있다. 중간처리 메소드들을 연이어 이어붙어 원하는 로직을 반복해서 사용할 수 있다.

1. filter()
- filter 메소드의 인자로 받은 람다식이 참이면 Optional 객체를 그대로 통과시키고 거짓이면 Optional.empty()를 반환해서 추가로 처리가 안되도록 한다.
```java
Optinal.of("ABCD").filter(v -> v.startsWith("AB")).orElse("Not AB");
// filter(변수 -> 변수.조건)
// 위 코드는 조건에 부합하므로 ABCD return 부합하지 않으면 orElse로 적은 Not AB가 return 된다.

Optional<String> opStr1 = Optional.of("first string");
Optional<String> opStr2 = Optional.of("second string");
Optional<String> filtered1 = opStr1.filter(s -> s.contains("first"));
Optional<String> filtered2 = opStr2.filter(s -> s.contains("first"));
filtered1.ifPresent(System.out::println); // Optional 객체 filtered1 출력
filtered2.ifPresent(System.out::println); // Optional.empty() 이므로 아무것도 출력되지 않는다.
```
  
2. map()
- Optional 객체의 값에 어떤 수정을 가해서 다른 값으로 변경하는 메서드
```java
Order order = getOrder(); // 

Optional.ofNullable(order)
			.map(Order::getMember)
			.map(Member::getAddress)
			.map(Address::getCity)
			.orElse("Gwon");
// 혹시 Order 객체가 null인 경우를 대비하여 of() 대신에 ofNullable()을 사용했습니다.
// map() 메소드의 연쇄 호출을 통해서 Optional 객체를 3번 변환하였습니다. 매 번 다른 메소드 레퍼런스를 인자로 넘겨서 Optional에 담긴 객체의 타입을 바꿔주었습니다. (Optional<Order> -> Optional<Member> -> Optional<Address> -> Optional<String>)
// 마무리 작업으로 orElse() 메소드를 호출하여 이 전 과정을 통해 얻은 Optional이 비어있을 경우, orElse() 값이 세팅됩니다.
``` 

## 🥎 값을 return하는 메소드
- 중간 메소드들은 Optional 객체를 return해서 메소드 체인으로 사용할 수 있는 반면, 값을 return해서 메소드 체인을 끝내는 메소드들도 있다.  

1. isPresent()
- Optional 객체의 값이 있으면 true 없으면 false를 return 해준다.
```java
Optional<String> option = Optional.ofNullable("asd");
System.out.println(option.isPresent()); // true
```  
  
2. ifPresent()
- ifPresent() 메소드는 람다식을 인자로 받아, 값이 존재할 때 그 값에 람다식을 적용해준다. 만약 Optoinal 객체에 값이 없다면 람다식이 실행되지 않는다.
```java
Optional<String> option = Optional.ofNullable("ifTest");
option.ifPresent(v -> System.out.print(v)); //ifTest 출력
```

3. get()
- Optional 객체가 가지고 있는 value 값을 꺼내온다. 만약 Optional 객체에 값이 없다면, NoSuchElementException이 발생된다.
```java
Optional<String> option = Optional.ofNullable("getTest");
System.out.print(option.get()); // getTest 출력
```  
  
4. orElse()
- 최종적으로 orElse가 실행될 때 Optional 객체가 비어있었다면, orElse() 메소드에 지정된 값이 기본값으로 return 된다.
```java
Optional<String> option;
System.out.println(option); // Optional.empty
System.out.println(option.orElse("Else Test")); // Else Test
```
5. orElseGet()
- 중간처리 메소드들을 거치면서 혹은 원래 Optional 객체가 비어있었다면, orElseGet() 메소드의 인자롤 입력되어 있는 Supplier 함수를 적용하여 객체를 얻어온다.
```java

Optional<String> test = Optional.of("XYZ").filter(v -> v.startsWith("AB")).orElseGet(() -> "NotAB"); // NotAB return

// orElse 차이점 - 테스트 1
String elseName = Optional.ofNullable("HoHo").orElse(test());
System.out.println(elesName);
String elseGetName = Optional.ofNullable("NoNo").orElseGet(()->test());
System.out.println(elseGetName);

private String test() {
    System.out.println("Test Print");
    return "i`m Test";
}
// 결과
// Test Print
// HoHo
// NoNo

// orElse 차이점 - 테스트 2
String a = null;
String elseName = Optional.ofNullable(a).orElse(test());
System.out.println(elesName);
String elseGetName = Optional.ofNullable(a).orElseGet(()->test());
System.out.println(elseGetName);

private String test() {
    System.out.println("Test Print");
    return "i`m Test";
}
// 결과
// Test Print
// i`m Test
// Test Print
// i`m Test
```
> 결론 : **orElse**에 매개변수로 메소드가 넣으면 Optional의 객체의 값이 null이든 아니든 해당 메소드를 무조건 실행한다. -> orElse문의 Value는 메모리상에 존재한다고 가정하기 때문에  
**orElseGet**는 null일 경우 Supplier를 호출하는 것이므로 null이 아니면 Supplier를 호출하지 않는다.  
  
6. orElseThrow()
- 해당 메소드에 도착했을 때 Optional 객체가 비어있었다면, Supplier 함수를 싱행해 예외를 발생시킨다. 
```java
Optional<String> option = Optional.empty();
option.orElseThrow(()->new NoClassDefFoundError()); // NoclassDefFoundError 에러 발생
option.orElseThrow(IndexOutOfBoundsException::new); // IndexOutOfBoundsException 에러 발생

```
  
## 🧵 Java9에서 추가된 메소드들
1. or()
- 중간처리 메소드로 OrElse(), orElseGet()과 비슷하지만 Optional 객체를 return 한다.
- 메소드 체인 중간에서 Optional empty()가 되었을 때, Optional.empty()대신 다른 Optional객체를 만들어서 뒤쪽으로 념겨준다.
```java
String a = null;        
System.out.print(Optional.ofNullable(a).or(()-> Optional.ofNullable("orCheck")).orElse("Null check")); 
// orCheck 출력
```
2. ifPresentOrElse()
- 최종적으로 값을 반환하는 메소드다. ifPresent() 메소드와 유사하지만 인자를 하나 더 받는다. 
- 첫번 째 인자로 받은 람다식은 Optional 객체에 값이 존재하는 경우 실행된다. 
- 두번째 인자로 받은 람다식은 Optional 객체가 비어있을 때 실행된다.
```java
Optional.ofNullable("test").ifPresentOrElse(value -> System.out.println(value+"Not empty"), () -> System.out.println("null"));
// test Not empty 출력
```
3. stream()
- 중간처리 연산자로 Java8에서 Optional 객체가 바로 스트림 객체로 전환 되지 않아 불편했던 부분을 해소시켜 줍니다.
```java
// 메서드 시그니처
public Stream<T> stream();
// 예제
List<String> result = List.of(1, 2, 3, 4)
    .stream()
    .map(val -> val % 2 == 0 ? Optional.of(val) : Optional.empty())
    .flatMap(Optional::stream)
    .map(String::valueOf)
    .collect(Collectors.toList());
System.out.println(result); // print '[2, 4]'

// 예제는 리스트에서 일부 값만 추출하고 스트림으로 변환한 뒤 다시 리스트로 수집하는 과정을 보여줍니다.
```

## 🛒 Java 10
1. orElseThrow()
- Java8의 orElseThrow와 동일하지만 인자를 받지 않는다는 차이가 있다.
```java
Optional<String> option = Optional.empty();
option.orElseThrow(()->new NoClassDefFoundError()); // NoclassDefFoundError 에러 발생
option.orElseThrow(); // NoSuchElementException: No value present
```