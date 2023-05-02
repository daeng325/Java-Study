# item43. 람다보다는 메소드 참조를 사용하라.

## 메소드 참조

앞에서 익명 클래스보다는 람다를 사용하라는 말이 있었는데, 자바에서는 함수 객체를 심지어 람다보다도 더 간결하게 만드는 방법이 있다. 그게 바로 메소드 참조(method reference)이다.&#x20;

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

깔끔해 보이는 코드지만 아직도 거추장스러운 부분이 남아 있다. 매개변수인 count와 incr은 크게 하는 일 없이 공간을 꽤 차지한다. 사실 이 람다는 두 인수의 합을 단순히 반환할 뿐이다. 자바 8이 되면서 Integer 클래스(와 모든 기본 타입의 박싱 타입)는 이 람다와 기능이 같은 정적 메소드 sum을 제공하기 시작했다.&#x20;

```java
map.merge(key, 1, Integer::sum);
```

<details>

<summary>Java 8 Map Improvements</summary>

```java
public void mapExample() {
    /**
     * 계산 패턴 개선
     */
    List<String> lines = new ArrayList<>();
    Map<String, byte[]> dataToHash = new HashMap<>();

    lines.forEach(line ->
            dataToHash.computeIfAbsent(line, // line은 맵에서 찾을 키
                    this::calculateDigest)); // 키가 존재하지 않으면 동작을 실행


    /**
     * computeIfAbsent 를 어떻게 여기 활용할 수 있을까?
     */
    Map<String, List<String>> friendsToMovies = new HashMap<>();
    String friend = "Raphael";
    List<String> movies = friendsToMovies.get(friend);
    if (movies == null) {
        movies = new ArrayList<>();
        friendsToMovies.put(friend, movies);
    }
    movies.add("Star wars");
    System.out.println(friendsToMovies);

    // 개선된 코드
    friendsToMovies.computeIfAbsent("Rahael", name -> new ArrayList<>())
            .add("Star wars");

    /**
     * 삭제 패턴 개선
     */
    Map<String, String> favouriteMovies = new HashMap<>();
    String key = "Raphael";
    String value = "Jack Reacher 2";
    if (favouriteMovies.containsKey(key) &&
            Objects.equals(favouriteMovies.get(key), value)) {
        favouriteMovies.remove(key);
    }
    else {
        //...
    }

    // 개선된 코드
    favouriteMovies.remove(key, value);

    /**
     * 교체 패턴
     */
    Map<String, String> favouriteMovies = new HashMap<>();
    favouriteMovies.put("Raphael", "Star Wars");
    favouriteMovies.put("Olivia", "james bond");
    favouriteMovies.replaceAll((key, movie) -> movie.toUpperCase());
    System.out.println(favouriteMovies); // {Olivia=JAMES BOND, Raphael=STAR WARS}

    favouriteMovies.replace("Raphael", "Harry Potter");
    favouriteMovies.replace("Olivia", "Harry Potter", "james bond"); // 변화 없음
    System.out.println(favouriteMovies); // {Olivia=JAMES BOND, Raphael=Harry Potter}

    /**
     * putAll, merge
     */
    Map<String, String> family = new HashMap<String, String>() {{
        put("Teo", "Star Wars");
        put("Cristina", "James Bond");
    }};
    Map<String, String> friends = new HashMap<String, String>() {{
        put("Raphael", "Star Wars");
    }};
    Map<String, String> everyone = new HashMap<>(family);
    everyone.putAll(friends);
    System.out.println(everyone); // {Cristina=James Bond, Raphael=Star Wars, Teo=Star Wars}
    // 해당 코드는 중복된 키가 없으면 잘 동작함. 값을 좀 더 유연하게 합쳐야 한다면?

    friends.put("Cristina", "Matrix");
    Map<String, String> everyone2 = new HashMap<>(family);
    friends.forEach((k, v) ->
            everyone2.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
    System.out.println(everyone2); // {Raphael=Star Wars, Cristina=James Bond & Matrix, Teo=Star Wars}

    // merge를 이용한 초기화 검사
    Map<String, Long> moviesToCount = new HashMap<>();
    String movieName = "JamesBond";
    Optional<Long> count = Optional.ofNullable(moviesToCount.get(movieName));
    if (!count.isPresent()) {
        moviesToCount.put(movieName, 1L);
    } else {
        moviesToCount.put(movieName, count.get() + 1);
    }

    // 개선된 코드
    moviesToCount.merge(movieName, 1L, (key, cnt) -> cnt + 1L); // 1L: 키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 값이 없거나 키에 널 값이 연관되어 있다면 이 값을 키와 연결
    // 키의 반환 값이 널이므로 처음에는 1이 사용된다. 그 다음부터는 값이 1로 초기화되어 있으므로 BiFunction을 적용해 값이 증가된다.


    /**
     * 퀴즈 8-2
     */
    Map<String, Integer> moviesTest = new HashMap<String, Integer>() {{
        put("JamesBond", 20);
        put("Matrix", 15);
        put("Harry Potter", 5);
    }};
    moviesTest.entrySet().removeIf(entry -> entry.getValue() < 10);
    System.out.println(moviesTest); // {Matrix=15, JamesBond=20}

}
```

</details>

## 메소드 참조의 장단점

* 장점: 매개변수 수가 늘어날수록 메소드 참조로 제거할 수 있는 코드 양도 늘어난다.&#x20;
* 단점: 오히려 람다에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 한다. \
  유지보수성 ↓, 가독성 ↓

### 메소드 참조를 쓰면 좋은 경우

그렇더라도 메소드 참조를 사용하는 편이 보통은 더 짧고 간결하니까, 람다가 너무 길어지면 람다로 작성할 코드를 새로운 메소드에 담은 다음, 람다 대신 그 메소드 참조를 사용하는 방법도 많이 사용한다. 메소드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서로 남길 수도 있다.

### 메소드 참조보다 그냥 람다로 두는 게 좋은 경우

&#x20;IDE들은 람다를 메소드 참조로 대체하라고 권할 것이다. IDE의 권고를 따르는 게 보통은 이득이지만, 항상 그런 것은 아니다. 때로는 람다가 메소드 참조보다 간결할 때도 있다.&#x20;

&#x20;   \--> 주로 메소드와 람다가 같은 클래스에 있을 때 그렇다.&#x20;

Example 1)&#x20;

```java
// 아래 코드가 GoshThisClassNameIsHumongous 클래스 내에 있다면?
service.execute(GoshThisClassNameIsHumongous::action);

service.execute(()-> action()); // 이게 더 짧다.
```

위  예시에서 메소드 참조 쪽은 더 짧지도, 더 명확하지도 않다. 따라서 람다 쪽이 낫다.&#x20;

Example 2)

`java.util.function` 패키지가 제공하는 Generic static factory method인 `Function.identity()`를 사용하기보다는 똑같은 기능의 람다`(x-> x)`를 직접 사용하는 편이 코드도 짧고 명확하다.&#x20;



## 메소드 참조의 유형

다음은 이상의 다섯 가지 메소드 참조를 정리한 표이다.&#x20;

| Method Reference | Example                | Lambda                                                        |
| ---------------- | ---------------------- | ------------------------------------------------------------- |
| 정적               | Integer::parseInt      | str -> Integer.parseInt(str);                                 |
| 한정적(인스턴스)        | Instant.now()::isAfter | <p>Instant then = Instant.now();<br>t -> then.isAfter(t);</p> |
| 비한정적(인스턴스)       | String::toLowerCase    | str -> str.toLowerCase();                                     |
| 클래스 생성자          | TreeMap\<K,V>::new     | () -> new TreeMap\<K,V>();                                    |
| 배열 생성자           | int\[]::new            | len -> new int\[len]                                          |

인스턴스 메소드 참조 ~~(말 드럽게 어려움)~~

1. 수신 객체 (receiving object; 참조 대상 인스턴스)를 특정하는 한정적 (bound) 인스턴스 메소드 참조
   1. 근본적으로 정적 참조와 비슷하다. \
      즉, 함수 객체가 받는 인수와 참조되는 메소드가 받는 인수가 똑같다.
2. 수신 객체를 특정하지 않는 비한정적 (unbound) 인스턴스 메소드 참조
   1. 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다. \
      이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메소드 선언에 정의된 매개변수들이 뒤따른다.
   2. 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰인다. (item45)



unbound receiver 아이디어

* 람다의 매개변수 중 하나로 제공될 객체의 메소드를 참조하는 것

```java
(String s) -> s.toUpperCase()

String::toUpperCase
```

bound receiver 아이디어

* 이미 존재하는 외부 객체의  메소드를 참조하는 것

```java
() -> expensiveTransaction.getValue()

expensiveTransaction::getValue
```



{% code title="Method Reference three usage" %}
```java
// 1. static
(args) -> ClassName.staticMethod(args)

ClassName::staticMethod

// 2. unbound
(arg0, rest) -> arg0.instanceMethod(rest)

ClassName::instanceMethod (arg0 is of type ClassName)

// 3. bound
(args) -> expr.instanceMethod(args)

expr::instanceMethod
```
{% endcode %}

{% hint style="success" %}
💡메소드 참조 쪽이 짧고 명확하다면 메소드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.
{% endhint %}



\+) 추가

보통 람다로 할 수 없는 일이라면 메소드 참조로도 할 수 없다. 여기서 애매한 예외가 존재하는데, 제네릭 함수 타입(generic function type) 구현이 유일한 예외이다. (람다로는 불가능하나 메소드 참조로는 가능)

함수형 인터페이스의 추상 메소드가 제네릭일 수 있듯이, 함수 타입도 제네릭일 수 있다. 다음의 인터페이스 계층 구조를 생각해 보자.&#x20;

```java
interface G1 {
    <E extends Exception> Object m() throws E;
}

interface G2 {
    <F extends Exception> String m() throws Exception;
}

interface G extends G1, G2 {
}

// Functional Interface G를 함수 타입으로 표현하면 다음과 같다.
<F extends Exception> () -> String throws F
```

함수형 인터페이스를 위한 제네릭 함수 타입은 메소드 참조 표현식으로는 구현할 수 있지만, 람다식으로는 불가능하다. 제네릭 람다식이라는 문법이 존재하지 않기 때문이다.
