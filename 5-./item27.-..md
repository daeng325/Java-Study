# item27. 비검사 경고를 제거하라.

제네릭을 지원하기 전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. 누군가 실수로 엉뚱한 타입의 객체를 넣어두면 런타임에 형변환 오류가 나곤 했다.&#x20;

반면, 제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에 알려주게 된다. 그래서 컴파일러는 알아서 형변환 코드를 추가할 수 있게 되고, 엉뚱한 타입의 객체를 넣으려는 시도를 컴파일 과정에서 차단하여 더 안전하고 명확한 프로그램을 만들어 준다.&#x20;

꼭 컬렉션이 아니더라도 이러한 이점을 누릴 수 있으나, 코드가 복잡해진다는 단점이 따라온다. 이번 장에서는 제네릭의 이점을 최대로 살리고 단점을 최소화하는 방법을 이야기한다.&#x20;



## 비검사 경고란?

`warning : [unchecked]`를 말하며, casting 할 때 검사를 하지 않았다고 뜨는 경고이다.

제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것이다.&#x20;

* 비검사 형변환 경고
* 비검사 메소드 호출 경고&#x20;
* 비검사 매개변수화 가변인수 타입 경고
* 비검사 변환 경고
* ...

제네릭에 익숙해질수록 마주치는 경고 수는 줄겠지만 새로 작성한 코드가 한 번에 개끗하게 컴파일되리라 기대하지는 말자.&#x20;

대부분의 비검사 경고는 쉽게 제거할 수 있다. 코드를 다음처럼 잘못 작성했다고 해보자.&#x20;

```java
Set<String> exaltation = new HashSet();
```

그러면 컴파일러는 무엇이 잘못됐는지 천천히 설명해줄 것이다. (javac 명령줄 인수에 -Xlint:uncheck 옵션을 추가해야 한다.)

<details>

<summary>IntelliJ에서 java compiler 인수 추가하는 방법</summary>

*

    <figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

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

