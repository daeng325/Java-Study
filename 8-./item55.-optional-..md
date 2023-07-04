# item55. Optional 반환은 신중히 하라.

자바8 이전 메소드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지

1. 예외를 던진다.
   1. 예외는 진짜 예외적인 상황에서만 사용해야 한다. (item 69)
   2. 예외를 생성할 때 스택 추적 전체를 캡처해서 비용 up
2. (반환 타입이 객체 참조라면) null을 반환한다.
   1. 별도의 null 처리 코드 추가 필요
   2. NullPointerException 발생 가능 (null을 반환하게 한 실제 원인과는 전혀 상관없는 코드에서)



자바8부터 또 하나의 선택지인 Optional\<T>가 생겼다.&#x20;

Optional\<T>는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 아무것도 담지 않은 옵셔널은 '비었다'고 하고, 반대로 어떤 값을 담은 옵셔널은 '비지 않았다'고 한다.

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional\<T>를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메소드가 만들어진다. Optioanl을 반환하는 메소드는 예외를 던지는 메소드보다 유연하고 사용하기 쉬우며, null을 반환하는 메소드보다 오류 가능성이 작다.&#x20;



{% code title="item30 코드 " %}
```java
public static <E extends Comparable<E>> E max (Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("Empty Collection");
    }
    
    E result = null;
    for (E e: c) {
        if (result == null || e.compareTo(result)>0) {
            result = Objects.requireNonNull(e);
        }
    }
    
    return result;
}
```
{% endcode %}

item30에서 위 예시는 c가 Empty일 때 IllegalArgumentException을 던지니, Optional\<E>를 반환하는 편이 더 낫다고 이야기 했었다.&#x20;



```java
public static <E extends Comparable<E>> Optional<E> max (Collection<E> c) {
    if (c.isEmpty()) {
        Optional.empty();
    }

    E result = null;
    for (E e: c) {
        if (result == null || e.compareTo(result)>0) {
            result = Objects.requireNonNull(e);
        }
    }

    return Optional.of(result);
}
```

Optional을 반환하도록 구현하기는 어렵지 않다. 적절한 static factory method를 사용해 Optional을 생성해주기만 하면 된다.&#x20;

* 빈 Optional반환 :  Optional.empty()
* 값이 든 Optional 반환: Optional.of(value), Optional.ofNullable(value)

{% hint style="info" %}
Optional을 반환하는 메소드에서는 절대 null을 반환하지 말자. Optional을 도입한 취지를 완전히 무시하는 행위이다.
{% endhint %}



null을 반환하거나 예외를 던지는 대신 Optional 반환을 선택해야 하는 기준은 무엇인가?

**Optional은 Checked Exception과 취지가 비슷하다(item 71).** 즉, 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. Unchecked Exception을 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔찍한 결과로 이어질 수 있다. 하지만 Checked Exception을 던지면 클라이언트에서는 반드시 이에 대처하는 코드를 작성해야 한다.&#x20;



메소드가 Optional을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다.&#x20;

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
// 기본값을 정해둘 수 있다.
```

```java
Toy myToy = max(tyos).orElseThrow(TemperTantrumExeption::new);
// 원하는 예외를 던질 수 있다
```

```java
Element lastNobelGas = max(Elements.NOBEL_GASES).get();
// 항상 값이 채워져 있다고 가정한다.
// 다만, 잘못 판단해서 값이 채워져 있지 않으면 NoSuchElementException 발생
```

```java
orElseGet(Supplier<? extends T> other)
// Optional에 값이 없을 때만 Supplier가 실행됨.
// 기본값을 설정하는 비용이 아주 커서 부담이 될 경우 사용 
```

```java
ifPresent(Consumer<? super T> consumer)
// 값이 존재할 때 인수로 넘겨준 동작 실행, 값이 없으면 아무 일도 일어나지 않는다.
```

<pre class="language-java"><code class="lang-java"><strong>ifPresentOrElse(Consumer&#x3C;? super T> action, Runnable emptyAction)
</strong>// Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다.
// 자바9부터 가능
</code></pre>

그 외에 filter, map, flatMap, isPresent 메소드들이 있다.&#x20;

{% code overflow="wrap" %}
```java
// 1. isPresent
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 pid: "+ (parentProcess.isPresent()? 
        String.valueOf(parentProcess.get().pid() : "N/A");
        
// 2. 1을map으로 대체 가능
System.out.println("부모 pid: "+ 
        ph.parent().map(h-> String.valueOf(h.pid()).orElse("N/A");

/**
* 3. Filter
* - Optional 객체가 값을 가지며 Predicate와 일치하면 filter 메소드는 그 값을 갖는 Optional을 반환, 그렇지 않으면 빈 Optional 객체를 반환
*/

// 기존
Insurance insurance1 = new Insurance("CambridgeInsurance");
if (insurance1 != null && "CambridgeInsurance".equals(insurance1.getName())) {
   System.out.println("ok");
}

// Optional filter() 사용
Optional<Insurance> optionalInsurance = Optional.ofNullable(insurance1);
optionalInsurance.filter(ins -> "CambridgeInsurance".equals(ins.getName()))
       .ifPresent(x-> System.out.println("ok"));
       
/** 
* 4.flatMap
* 평준화 과정: 이론적으로 두 Optional을 합치는 기능을 수행, 둘 중 하나라도 null이면 빈 Optional을 생성
* 값이 있으면 제공된 Optional을 가진 매핑 함수를 적용하여 그 결과를 반환하고, 그렇지 않으면 빈 Optional을 반환
**/

public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
}


Person person = new Person();
Optional<Person> optPerson = Optional.of(person);

String nameWithFlatMap = optPerson.flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("UnKnown"); // 결과 Optional이 비어있으면 기본값 사용
```
{% endcode %}

Stream을 사용한다면 Optional들을 Stream\<Optional\<T>>로 받아서, 그 중 비어있지 않은 옵셔널들에서 값을 뽑아 Stream\<T>에 건네 담아 처리하는 경우가 많다.

자바 8에서는 다음과 같이 구현

```java
streamOfOptionals    
    .filter(Optional::isPresent)
    .map(Optional::get)
```

자바9에서는 Optioanl에 stream() 메소드 추가

* Optional에 값이 있으면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다. 이를 Stream의 flatMap 메소드와 조합하면 앞의 코드를 명료하게 바꿀 수 있음.

```java
streamOfOptionals
    .flatMap(Optional::stream)

// Example 1
List<Person> persons = new ArrayList<>();
persons.stream()
        .map(Person::getCar)
        .map(optCar -> optCar.flatMap(Car::getInsurance))
        .map(optIns -> optIns.map(Insurance::getName))
        .flatMap(Optional::stream) // Stream<Optional<String>>을 현재 이름을 포함하는 Stream<String>으로 변환
        .collect(Collectors.toSet());


Stream<Optional<String>> stream = ...

Set<String> result = stream.filter(Optional::isPresent)
                             .map(Optional::get)
                             .collect(toSet());
```

