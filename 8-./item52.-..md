# item52. 다중정의는 신중히 사용하라.

## 다중정의(overloading, 오버로딩)

```java
// 컬렉션을 집합, 리스트, 그 외로 구분하고자 만든 프로그램
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> list) {
        return "리스트";
    }

    public static String classify(Collection<?>c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c: collections) {
            System.out.println(classify(c));
        }
    }
}
```

<details>

<summary>위 프로그램은 무엇을 출력할까?</summary>

"집합", "리스트", "그 외"를 차례로 출력할 것 같지만, 실제로 수행해보면 "그 외"만 세 번 연달아 출력한다.&#x20;

</details>



이유가 뭘까?&#x20;

**다중정의(overloading, 오버로딩)**된 세 classify 중 어느 메소드를 호출할지가 컴파일타임에 정해지기 때문이다.  컴파일타임에는 for문 안의 c는 항상 Collection\<?> 타입이다. 런타임에는 타입이 매번 달라지지만, 호출할 메소드를 선택하는 데에는 영향을 주지 못한다.&#x20;



### 오버라이딩과 오버로딩

재정의(overriding)한 메소드는 동적으로 선택되고, 다중정의(overloading)한 메소드는 정적으로 선택된다.&#x20;

* 메소드를 오버라이딩 했다면 해당 객체의 런타임 타입이 어떤 메소드를 호출할지의 기준이 된다.  컴파일 타임에 인스턴스의 타입이 무엇이었냐는 상관 없다.&#x20;

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override
    String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override
    String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = Arrays.asList(new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine: wineList) {
            System.out.println(wine.name());
        }
    }
}
```

* 메소드를 오버로딩 했다면 객체의 런타임 타입은 전혀 중요치 않다. 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에 의해 이뤄진다.&#x20;



맨 위 코드에서 의도한 대로 표출되게 하려면 어떻게 해야 할까?

* (정적 메소드를 사용해도 좋다면) CollectionClassifier의 모든 classify 메소드를 하나로 합친 후 instanceof로 명시적으로 검사하면 말끔히 해결된다.&#x20;

```java
public static String classify(Collection<?> c) {
        return c instanceof Set ? "집합" :
                c instanceof List ? "리스트" : "그 외";
}
```



## 다중정의를 올바르게 사용하려면?

### 1. 매개변수 수가 같은 다중정의는 아예 만들지 말자.

이처럼 다중정의가 혼동을 일으키는 상황을 피해야 한다. 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자. 다중정의하는 대신 메소드 이름을 다르게 지어주는 길도 항상 열려 있으니 말이다.&#x20;

* ObjectOutputStream 클래스를 살펴보자. 이 클래스의 write 메소드는 모든 기본 타입과 일부 참조 타입용 변형을 가지고 있다.  그런데 오버로딩 된 형태가 아니라 writeBoolean(boolean), writeInt(int), writeLong(long)과 같은 형식이다.  이렇게 되면 read 메소드의 이름과 짝을 맞추기 좋게 된다.  ObjectInputStream도 readBoolean(), readInt(), readLong() 같은 형태로 되어 있다.



(본인 의견)

위처럼 아예 오버로딩을 하지 않는다는 건 너무 보수적인 경우 같음. 오버로딩된 함수의 argument중 하나가 다른 것들을 커버할 수 있는 상황이라면 (예시처럼 하나가 Collection이고 다른 것들이 그 Collection을 상속한 경우) 아예 따로 메소드명을 달리 해서 빼는 게 좋다는 것으로만 인식하면 될 듯.&#x20;



### 2. 가변인수를 사용하는 메소드는 다중정의를 하지 말자.

가변인수를 사용하는 메소드라면 다중정의를 아예 하지 말아야 한다 (item 53에 예외 기술).&#x20;



### 3. 근본적으로 다른 매개변수 타입을 사용하자.

한편, 생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 된다. 하지만 정적 팩토리라는 대안을 활용할 수 있는 경우가 많다 (item 1). 또한 생성자는 오버라이딩 할 수 없으니 오버로딩과 혼용될 걱정도 없다. 그래도 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우를 완전히 피해갈 수는 없을 테니, 이를 대비한 안전 대책을 배워 보자.



#### 매개변수 중 하나 이상이 근본적으로 다르다면 헷갈릴 일이 없다.&#x20;

매개변수 수가 같은 오버로딩 메소드가 많더라도, 그 중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분된다면 헷갈릴 일은 없을 것이다. 매개변수 중 하나 이상이 "근본적으로 다르다(radically different)"는 건 두 타입의 (null이 아닌) 값을 서로 어느쪽으로든 형변환할 수 없다는 뜻이다.&#x20;



## 다중정의 방해 요소

### 1.  오토박싱

JDK1.4 까지는 모든 기본 타입이 모든 참조 타입과 근본적으로 달랐지만, JDK1.5에서 오토박싱이 도입되면서 평화롭던 시대가 막을 내렸다.&#x20;

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }

        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }

        System.out.println(set + " " + list); // [-3,-2,-1] [-3,-2,-1] ??
    }
}
```

위 코드는 "\[-3, -2, -1] \[-2, 0, 2]"를 출력한다. 이게 무슨 일일까?

* set.remove(i)의 시그니처는 `remove(Object)`다. 다중정의된 다른 메소드가 없어서 기대한 대로 동작하여 집합에서 0이상의 수들을 제거한다.
* list.remove(i)는 다중 정의된 `remove(int index)`를 선택한다. remove는 '지정한 위치'의 원소를 제거하는 기능을 수행하게 된다.&#x20;
  * `list.remove((Integer) i)`
  * `list.remove(Integer.valueOf(i))`



### 2.  람다와 메소드 참조

JDK1.8에서 도입한 람다와 메소드 참조 역시 다중정의 시의 혼란을 키웠다.&#x20;

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번. ExecutorService의 submit 메소드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

2번만 컴파일 오류가 난다.&#x20;

* Reference to 'println' is ambiguous, both 'println()' and 'println(boolean)' match

넘겨진 인수는 System.out::println으로 똑같고, 양쪽 모두 Runnable을 받는 형제 메소드를 다중정의하고 있다. 그런데 왜 한쪽만 실패할까? 원인은 바로 submit 다중정의 메소드 중에는 Callable\<T>를 받는 메소드도 있다는 점이다.&#x20;

근데 컴파일 에러는 왜 위처럼 날까? 그건 만약 prinln이 다중정의 없이 단 하나만 존재했다면 이 submit 메소드 호출이 제대로 컴파일됐을 것이기 때문이다. ~~--> 이건 왜 그런지 아래 설명했는데 뭔 소린지 모르겠음~~

> System.out::println은 부정확한 메소드 참조(inexact method reference)다.
>
> "암시적 타입 람다식(implicitly typed lambda expression)이나 부정확한 메소드 참조 같은 인수 표현식은 목표 타입이 선택되기 전에는 그 의미가 정해지지 않기 때문에 적용성 테스트(applicability test) 때 무시된다."

컴파일러 제작자를 위한 설명이니 무슨 말인지 이해되지 않더라도 그냥 넘어가자고 함. 핵심은 다중정의된 메소드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을 때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다는 것이다. 따라서 메소드를 오버라이딩할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다.&#x20;



## 다중정의를 주의하지 못한 사례

이번 아이템에서 설명한 지침들을 어기고 싶을 때도 있을 것이다. 이미 만들어진 클래스가 끼어들면 특히 더 그렇다.

예를 들어 String은 자바 4 시절부터 contentEquals(StringBuffer) 메소드를 가지고 있었다. 그런데 자바 5에서 StringBuffer, StringBuilder, String, CharBuffer 등의 비슷한 부류의 타입을 위한 공통 인터페이스로 CharSequence가 등장하였고, 자연스럽게 String에도 CharSequence를 받은 contentEquals가 다중정의되었다.&#x20;

다행히 이 두 메소드는 같은 객체를 임력하면 완전히 같은 작업을 수행해주니 해로울 건 전혀 없다. 이처럼 어떤 다중정의 메소드가 불리는지 몰라도 기능이 똑같다면 신경 쓸 게 없다. 이렇게 하는 가장 일반적인 방법은 상대적으로 더 특수한 다중 정의 메소드에서 덜 특수한(더 일반적인) 다중정의 메소드로 일을 넘겨버리는 (forward) 것이다.&#x20;

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```



자바 라이브러리는 이번 아이템의 정신을 지켜내려 애쓰고 있지만, 실패한 클래스도 몇 개 있다. 예컨대 String 클래스의 valueOf(char\[])과 valueOf(Object)는 같은 객체를 건네더라도 전혀 다른 일을 수행한다. 이렇게 해야 할 이유가 없었음에도, 혼란을 불러올 수 있는 잘못된 사례로 남게 되었다.



{% hint style="info" %}
일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다.&#x20;

상황에따라, 특히 생성자라면 이 조언을 따르기가 불가능할 수 있다. 그럴 때는 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메소드가 선택되도록 해야 한다.&#x20;

이것이 불가능하다면, 예컨대 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메소드들이 모두 동일하게 동작하도록 만들어야 한다.&#x20;
{% endhint %}



