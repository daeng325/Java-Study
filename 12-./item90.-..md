# item90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라.

Serializable을 구현하기로 결정한 순간 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 된다. 버그와 보안 문제가 일어날 가능성이 커진다는 뜻이다. 하지만 이 위험을 크게 줄여줄 기법이 하나 있다. 바로 직렬화 프록시 패턴 (Serialization Proxy Pattern)이다.&#x20;



## 직렬화 프록시 패턴

먼저, 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시이다.&#x20;

### 직렬화 프록시 선조건&#x20;

* 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다. 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다. 일관성 검사나 방어적 복사도 필요 없다. 설계 상, 직렬화 프록시의 기본 직렬화 형태는 바깥 클래스의 직렬화 형태로 쓰기에 이상적이다.&#x20;
* 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다고 선언해야 한다.&#x20;



item50에서 작성했고 item88에서 직렬화한 Period 클래스를 예로 살펴보자. 다음은 이 클래스의 직렬화 프록시다. Period는 아주 간단하여 직렬화 프록시도 바깥 클래스와 완전히 같은 필드로 구성되었다.&#x20;

<pre class="language-java"><code class="lang-java">public class Period5 implements Serializable{
    private final Date start;
    private final Date end;

    public Period5(Date start, Date end) {
        /**
         * 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한다.
         * --> 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있다.
         */
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

    /**
     * 필드의 방어적 복사본을 반환한다.
     */
    public Date start(){
        return new Date(start.getTime());
    }

    /**
     * 필드의 방어적 복사본을 반환한다.
     */
    public Date end() {
        return new Date(end.getTime());
    }

    public String print() {
        return "Start: " + start + ", End: " + end;
    }
    
<strong>    // 직렬화 프록시 패턴용 writeReplace 메소드
</strong><strong>    private Object writeReplace() {
</strong><strong>        return new SerializationProxy(this);
</strong><strong>    }
</strong>
<strong>    // 직렬화 프록시 패턴용 readObject 메소드
</strong><strong>    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
</strong><strong>        throw new InvalidObjectException("프록시가 필요합니다.");
</strong><strong>    }
</strong>    
<strong>    private static  class SerializationProxy implements Serializable {
</strong><strong>        private final Date start;
</strong><strong>        private final Date end;
</strong><strong>
</strong><strong>        SerializationProxy(Period5 p) {
</strong><strong>            this.start = p.start();
</strong><strong>            this.end = p.end();
</strong><strong>        }
</strong><strong>
</strong><strong>        private Object readResolve() {
</strong><strong>            return new Period5(start, end); // public 생성자를 사용한다.
</strong><strong>        }
</strong><strong>        
</strong><strong>        private static final long serialVersionUID = 1234567890L; // 아무 값이나 상관 없다. (item 87)
</strong><strong>    }
</strong>
}



</code></pre>

### writeReplace 메소드&#x20;

이 메소드는 자바의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 SerializationProxy의 인스턴스를 반환하게 하는 역할을 한다. 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해 준다.&#x20;



### readObject 메소드

writeReplace 덕분에 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다. 하지만 공격자는 불변식을 훼손하고자 이런 시도를 해볼 수 있다. 이런 시도는 readObject 메소드를 바깥 클래스에 추가하면 가볍게 막아낼 수 있다.&#x20;



### readResolve 메소드&#x20;

마지막으로, 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메소드를 SerializationProxy 클래스에 추가한다. 이 메소드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.



여기서readResolve 메소드는 공개된 API만을 사용해 바깥 클래스의 인스턴스를 생성하고 있다.&#x20;

직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하는 기능을 제공하는데, 이 패턴은 직렬화의 이런 언어도단적 특성을 상당 부분 제거한다. 즉, 일반 인스턴스를 만들 때와 똑같은 생성자, 정적 팩터리, 혹은 다른 메소드를 사용해 역직렬화된 인스턴스를 생성하는 것이다. 따라서 역직렬화된 인스턴스가 해당 클래스의 불변식을 만족하는지 검사할 또 다른 수단을 강구하지 않아도 된다.&#x20;



## 직렬화 프록시 패턴의 장점&#x20;

방어적 복사처럼, 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해 준다. 앞서의 두 접근법과 달리, 직렬화 프록시는 Period의 필드를 final로 선언해도 되므로 Period 클래스를 진정한 불변(item 17)으로 만들 수도 있다. 어떤 필드가 직렬화 공격의 목표가 될지 고민하지 않아도 되며, 역직렬화 때 유효성 검사를 수행하지 않아도 된다.&#x20;



직렬화 프록시 패턴이 readObject에서의 방어적 복사보다 강력한 경우가 하나 더 있다. (가변 공격 대비)직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 동작한다. 실전에서 무슨 쓸모가 있나 싶겠지만, 쓸모가 있다.

EnumSet의 사례를 생각해 보자 (item 36). 이 클래스는 public 생성자 없이 정적 팩터리들만 제공한다. 클라이언트 입장에서는 이 팩터리들이 EnumSet 인스턴스를 반환하는 걸로 보이지만, 현재의 OpenJDK를 보면 열거 타입의 크기에 따라 두 하위 클래스 중 하나의 인스턴스를 반환한다. 열거 타입의 원소가 64개 이하면 RegularEnumSet을 사용하고, 그보다 크면 JumboEnumSet을 사용하는 것이다.

이제 원소 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하고 역직렬화하면 어떤 일이 벌어질지 알아보자. 처음 직렬화된 것은 RegularEnumSet 인스턴스다. 하지만 역직렬화는 JumboEnumSet 인스턴스로 하면 좋을 것이다. 그리고 EnumSet은 직렬화 프록시 패턴을 사용해서 실제로도 이렇게 동작한다.&#x20;



```java
// EnumSet의 직렬화 프록시 코드
private static class SerializationProxy <E extends Enum<E>>
        implements java.io.Serializable
{
    /**
     * The element type of this enum set.
     *
     * @serial
     */
    private final Class<E> elementType;

    /**
     * The elements contained in this enum set.
     *
     * @serial
     */
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(ZERO_LENGTH_ENUM_ARRAY); // new Enum<?>[0]
    }

    // instead of cast to E, we should perhaps use elementType.cast()
    // to avoid injection of forged stream, but it will slow the implementation
    @SuppressWarnings("unchecked")
    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType);
        for (Enum<?> e : elements)
            result.add((E)e);
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}


public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```



### 직렬화 프록시 패턴의 한계

1. 클라이언트가 멋대로 확장할 수 있는 클래스(item19)에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에는 적용할 수 없다. \
   이런 객체의 메소드를 직렬화 프록시의 readResolve 안에서 호출하려 하면 ClassCastException이 발생할 것이다. 직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어진 것이 아니기 때문이다.
3. Period 예를 실행하면 방어적 복사 때보다 14%가 느려진다. 성능 이슈 존재.



{% hint style="info" %}
제3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자. 이 패턴이 아마도 중요한 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법일 것이다.&#x20;
{% endhint %}
