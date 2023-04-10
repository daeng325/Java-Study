# item29. 이왕이면 제네릭 타입으로 만들라.

## 제네릭 타입이란?

제네릭 타입 - class\<T>, interface\<T>

* 타입을 파라미터로 가지는 클래스와 인터페이스

```java
public class Box<T> {
    private T t;
    public T get() { return t; }
    public void set(T t) { this.t = t;}
}

// T는 Box 클래스로 객체를 생성할 때 구체적인 타입으로 변경된다. 
Box<String> box1 = new Box<String>();
box1.set("hello");
String str = box1.get();

Box<Integer> box2 = new Box<Integer>();
box2.set(6);
int value = box2.get();
```

제네릭 메소드 - \<T, R> R method(T t)

* 파라미터 타입과 리턴 타입으로 타입 파라미터를 갖는 메소드&#x20;

```java
public <T> Box<T> boxing(T t) {...}

Box<Integer> box = <Integer>boxing(100); // 타입 파라미터를 명시적으로 Integer로 지정
Box<Integer> box = boxing(100); // 타입 파라미터를 Integer로 추정
```



## 일반 클래스를 제네릭 클래스로

일반 클래스를 제네릭 클래스로 만들어 보자. (ex. item7에서 썼던 Stack 클래스)

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        // 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.
        // 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.
        // elements 배열의 '활성 영역'(인덱스가 size보다 작은 원소들) 외의 참조들이 모두 여기 해당한다.
//        return elements[--size];

        // 참조 해제한 코드
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}

```

#### 클래스 선언에 타입 매개변수 추가

#### elements 타입 Object\[] -->  E\[] 로 변경

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
 //        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
        this.elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 오류!
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}

```

#### 컴파일 오류 고치기

<figure><img src="../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

item28 (배열보다는 리스트를 사용하라) 에서  설명한 것처럼, E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.&#x20;

* 주입식 교육의 폐해로 인해 그냥 머릿속에 '제네릭은 타입 정보가 런타임 시에 소거된다 (컴파일.시에 잡아줌).  이게 실체화가 불가능하다는 뜻.  그에 반해 배열은 런타임 시에 실체화가 됨.' 이라고 입력 박아버림.  정확한 이해는 쓰다보면 이해하겠지...



## 배열을 사용하는 코드를 제네릭으로 만들려 할 때&#x20;

&#x20;1\. Object 배열을 생성한 다음 제네릭 배열로 형변환

2\. elements 필드의 타입을 E\[]에서 Object\[]로 바꾸는 방법.

{% tabs %}
{% tab title="1." %}
```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 받는다.
    // 따라서 타입 안정성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[] 이다!
    @SuppressWarnings("unchecked")
    public Stack() {
 //        this.elements = new E[DEFAULT_INITIAL_CAPACITY];
        this.elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}

```
{% endtab %}

{% tab title="2." %}
```java
public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
//        this.elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
    ...
    
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size]; // 컴파일 오류!
        elements[size] = null;
        return result;
    }
    
    ...
}
```



<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

배열이 반환한 원소를 E로 형변환

```java
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
//        E result = (E) elements[--size];
        // E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.  우리가 직접 증명하고 경고를 숨길 수 있다.

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        @SuppressWarnings("unchecked") E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }
```
{% endtab %}
{% endtabs %}

현업에서는 첫 번재 방식을 더 선호하며 자주 사용한다고 한다. 첫 번째 방식은 제네릭 배열을 생성하지 말라는 제약을 대놓고 우회하는 방식인데, 이에 대한 장단점을 알아보자.



{% hint style="success" %}
:thumbsup:&#x20;

1.  가독성이 더 좋다.

    \--> 배열의 타입을 E\[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실하게 어필
2.  코드도 더 짧다.

    \--> 첫 번째 방식에서는 형변환을 배열 생성 시 단 한 번만 해 주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 형변환 필요 (Object 니까)
{% endhint %}

{% hint style="warning" %}
:thumbsdown:&#x20;

(E가 Object가 아닌 한) 배열의 런타임 타입이 컴파일 타입과 달라 힙 오염(heap pollution: item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라.)을 일으킨다.&#x20;

heap pollution이 마음에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다.&#x20;
{% endhint %}



## 핵심 정리

{% hint style="info" %}
💡 기존 타입 중 제네릭이어야 하는 게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.&#x20;

클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하다.
{% endhint %}

