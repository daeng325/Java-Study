# item55. Optional 반환은 신중히 하라.

자바8 이전 메소드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지

1. 예외를 던진다.
   1. 예외는 진짜 예외적인 상황에서만 사용해야 한다. (item 69)
   2. 예외를 생성할 때 스택 추적 전체를 캡처해서 비용 up
2. (반환 타입이 객체 참조라면) null을 반환한다.
   1. 별도의 null 처리 코드 추가 필요
   2. NullPointerException 발생 가능 (null을 반환하게 한 실제 원인과는 전혀 상관없는 코드에서)



자바8부터 또 하나의 선택지인 Optional\<T>가 생겼다.&#x20;

Optional\<T>는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. Optional은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션이다. Optional\<T>가 Collection\<T>를 구현하지는 않았지만, 원칙적으로는 그렇다.&#x20;

Optioanl을 반환하는 메소드는 예외를 던지는 메소드보다 유연하고 사용하기 쉬우며, null을 반환하는 메소드보다 오류 가능성이 작다.&#x20;



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

item30에서  위 예시는I  c가 Empty일 때 IllegalArgumentException을 던지니, Optional\<E>를 반환하는 편이 더 낫다고 이야기 했었다.&#x20;



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
* 값이 든 Optional 반환: Optional.of(value)





