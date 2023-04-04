# item27. 비검사 경고를 제거하라.

<details>

<summary>제네릭</summary>

제네릭 타입을 이용함으로써 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거할 수 있게 되었다. 메소드를 정의할 때 type을 parameter로 사용할 수 있도록 한다. 타입 파라미터는 코드 작성 시 구체적인 타입으로 대체되어 다양한 코드를 생성하도록 해준다.

## 제네릭의 이점

* 컴파일 시 강한 타입 체크를 할 수 있다.&#x20;
* 타입 변환(casting)을 제거한다.&#x20;
  * 비제네릭 코드는 불필요한 타입 변환을 하기 때문에 프로그램 성능에 악영향을 미친다.&#x20;

```java
List list = new ArrayList();
list.add("hello");
String str = (String) list.get(0);// 타입 변환을 해야 한다. 
```

```java
List<String> list = new ArrayList<String>();
list.add("hello");
String str = list.get(0); // 타입 변환을 하지 않는다. (프로그램 성능 향상)
```

## 와일드카드 타입 `(<?>, <? extends ...>, <? super ...>)`

코드에서 ?를 일반적으로 와일드카드(wildcard)라고 부른다. 제네릭 타입을 매개값이나 리턴 타입으로 사용할 때 구체적인 타입 대신에 와일드카드를 다음과 같이 세 가지 형태로 사용할 수 있다.&#x20;

```java
public class Course<T> {
    private String name;
    private T[] students;

    public Course(String name, int capacity) {
        this.name = name;
        this.students = (T[]) (new Object[capacity]); // 타입 파라미터로 배열을 생성하려면 new T[n] 형태로 배열을 생성할 수 없고 (T[]) (new Object[n]으로 생성해야 한다.)
    }

    public String getName() {
        return name;
    }

    public T[] getStudents() {
        return students;
    }
    
    public void add(T t) {
        // 배열에 비어 있는 부분을 찾아서 수강생을 추가하는 메소드
        for (int i = 0; i < students.length; i++) {
            if (students[i] == null) {
                students[i] = t;
                break;
            }
        }
    }
}

```

*

    <figure><img src="../.gitbook/assets/file.excalidraw (2).svg" alt=""><figcaption></figcaption></figure>
* `Course<?>`: Unbounded Wildcards (제한 없음)
  * 수강생은 모든 타입(Person, Worker, Student, HighStudent, Dog)이 될 수 있다.
* `Course<? extends Student>`: Upper Bounded Wildcards (상위 클래스 제한)
  * 수강생은 Student와 HighStudent만 될 수 있다.&#x20;
* `Course<? super Worker>`: Lower Bounded Wildcards (하위 클래스 제한)
  * 수강생은 Worker와 Person만 될 수 있다.&#x20;

### 제네릭 타입\<T>와 와일드 카드\<?>의 차이는?

* 제네릭 : 타입을 모르지만, 타입을 정해지면 그 타입의 특성에 맞게 사용한다.
* 와일드 카드 : 무슨 타입인지 모르고, 무슨 타입인지 신경쓰지 않는다. 타입을 확정하지 않고 가능성을 열어둔다.

특정 타입을 지정하여 타입에 따른 메소드를 사용하고 싶다면 제네릭 타입을 사용해야 한다.

</details>

## 비검사 경고란?

`warning : [unchecked]`를 말하며, casting 할 때 검사를 하지 않았다고 뜨는 경고이다.

제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것이다.&#x20;

* 비검사 형변환 경고
* 비검사 메소드 호출 경고&#x20;
* 비검사 매개변수화 가변인수 타입 경고
* 비검사 변환 경고
* ...

제네릭에 익숙해질수록 마주치는 경고 수는 줄겠지만 새로 작성한 코드가 한 번에 깨끗하게 컴파일 되리라 기대하지는 말자.&#x20;

대부분의 비검사 경고는 쉽게 제거할 수 있다. 코드를 다음처럼 잘못 작성했다고 해보자.&#x20;

```java
Set<String> exaltation = new HashSet();
```

그러면 컴파일러는 무엇이 잘못됐는지 천천히 설명해줄 것이다. (javac 명령줄 인수에 -Xlint:uncheck 옵션을 추가해야 한다.)

<details>

<summary>IntelliJ에서 java compiler 인수 추가하는 방법</summary>

*

    <figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

</details>

> ```
> java: unchecked conversion 
>     required: java.util.Set<java.lang.String> 
>     found: java.util.HashSet
> ```

> ```
> Unchecked assignment: 'java.util.HashSet' to 'java.util.Set<java.lang.String>'
> ```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

컴파일러가 알려준 대로 수정하면 경고가 사라진다. 사실 컴파일러가 알려준 타입 매개변수를 명시하지 않고, 자바7부터 지원하는 다이아몬드 연산자(<>)만으로 해결 가능하다. 그러면 컴파일러가 올바른 실제 타입 매개변수를 추론해준다.

```java
Set<String> exaltation = new HashSet<>();
```

{% hint style="success" %}
💡 할 수 있는 한 모든 비검사 경고를 제거하라.&#x20;

모두 제거한다면 그 코드는 타입 안정성이 보장된다. 즉, 런타임에 `ClassCastException`이 발생할 일이 없고, 여러분이 의도한 대로 잘 동작하리라 확신할 수 있다.
{% endhint %}



## `@SuppressWarnings`

{% hint style="warning" %}
💡 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 어노테이션을 달아 경고를 숨기자.
{% endhint %}

`@SuppressWarnings("unchecked")` 어노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 그러나 항상 가능한 한 좁은 범위에 적용하도록 하자(ex. 변수 선언, 아주 짧은 메소드, 혹은 생성자). 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안 된다.

한 줄이 넘는 메소드나 생성자에 달린 `@SuppressWarnings` 어노테이션을 발견하면 지역변수 선언 쪽으로 옮기자.

{% code title="ArrayList.toArray 메소드" %}
```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
{% endcode %}

> ```null
> ArrayList.java:305: warning: [unchecked] unchecked cast
> 	return (T[]) Arrays.copyOf(elements, size, a.getClassO);
>   required: T[]
>   found: Object []
> ```

어노테이션은 선언에만 달 수 있기 때문에 return문에는 `@SuppressWarnings`를 다는 게 불가능하다. 그렇다면 이제 메소드 전체에 달고 싶겠지만, 범위가 필요 이상으로 넓어지니 자제하자. 그 대신 반환값을 담을 지역변수를 하나 선언하고 그 변수에 어노테이션을 달아주자.

* 이를 위해 지역변수를 새로 선언하는 수고를 해야 할 수도 있지만, 그만한 값어치가 있을 것이다.&#x20;

{% code title="ArrayList.toArray 수정" %}
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
        @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
{% endcode %}

이 코드는 깔끔하게 컴파일 되고 비검사 경고를 숨기는 범위도 최소로 좁혔다.&#x20;

{% hint style="success" %}
💡 @SupperessWarnings("unchecked") 어노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.
{% endhint %}

