# item21. 인터페이스는 구현하는 쪽을 생각해 설계하라.

자바8 이전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메소드를 추가할 방법이 없었다. 인터페이스에 메소드를 추가하면 보통 컴파일 오류가 난다. (추가된 메소드가 이미 구현 객체에 존재한다면 오류가 나지 않겠지만 그럴 가능성은 아주 낮다.)

자바8에 와서 기존 인터페이스에 메소드를 추가할 수 있도록 디폴트 메소드를 소개했지만, 위험이 완전히 사라진 것은 아니다.&#x20;

1. Default Method가 뭔데?
2. 어떤 위험이 있길래?



## 디폴트 메소드란?

기존 인터페이스에 메소드를 추가할 수 있게 하기 위해 만들어짐.

인터페이스에 선언되지만 사실은 구현 객체가 가지고 있는 인스턴스 메소드라고 생각해야 함.

&#x20;   \-> 구현 객체가 있어야 사용 가능

{% code title="Interface - Jdk1.8" lineNumbers="true" %}
```java
public interface RemoteControl{
	// 상수(public static final)
	public int MAX_VOLUME=10;
	public int MIN_VOLUME=0;
	
	// 추상 메소드(public abstract)
	public void turnOn();
	public void turnOff();
	public void setVolume();
	
	// 디폴트 메소드(public default) -- Java 8부터 작성 가능
	default void setMute(boolean mute){
		if(mute){
			System.out.println("무음 처리합니다.");
		} else{
			System.out.println("무음 해제합니다.");
		}
	}
	
	// 정적 메소드(public static) -- Java 8부터 작성 가능 (디폴트 메소드와 달리 구현 객체가 없어도 인터페이스만으로 호출 가능)
	static void changeBattery(){
		System.out.println("건전지를 교환합니다.");
	}
}

```
{% endcode %}

불가능

```java
RemoteControl.setMute(true);
```

가능

```java
RemoteControl rc = new Television();
rc.setMute(true);
```

\-> 비록 Television에 setMute 메소드가 선언되지는 않았지만, Television 객체가 없다면 setMute()도 호출할 수 없다.

\-> default method도 구현 객체에서 오버라이드 가능하다.&#x20;



### 자바8에서 디폴트 메소드를 허용한 이유?

기존 인터페이스를 확장해서 새로운 기능을 추가하기 위해서이다.&#x20;

인터페이스에 다른 기능을 추가해야 하는데, 추상 메소드로 추가하면 그것을 구현한 구현객체들은 해당 추상메소드를 강제로 다 구현해야 하므로 코드 변경점이 많아진다.&#x20;



### 디폴트 메소드가 있는 인터페이스 상속

자식 인터페이스에서 디폴트 메소드를 활용하는 방법 3가지

1. 디폴트 메소드를 단순히 상속만 받는다.
2. 디폴트 메소드를 재정의(Override)해서 실행 내용을 변경한다.
3. 디폴트 메소드를 추상 메소드로 재선언한다.&#x20;



부모 인터페이스

```java
public interface ParentInterface{
	public void method1();
	public default void method2() {/*실행문*/}
}
```

{% tabs %}
{% tab title="1. 단순 상속" %}
```java
public interface ChildInterface1 extends ParentInterface{
	public void method3();
}
```

\--> ChildInterface1의 구현 객체: method1()과 method3()의 실체 메소드 필요

\--> ParentInterface의 method2() 호출 가능
{% endtab %}

{% tab title="2. 오버라이딩" %}
```java
public interface ChildInterface2 extends ParentInterface{
	@Override
	public default void method2(){/*실행문*/} // 재정의
	
	public void method3();
}
```

\--> ChildInterface2의 구현객체: method1()과 method3()의 실체 메소드 필요

\--> ChildInterface2에서 재정의한 method2() 호출 가능
{% endtab %}

{% tab title="3. 추상메소드로 재선언" %}
```java
public interface ChildInterface3 extends ParentInterface{
	@Override
	public void method2(); //추상 메소드로 재선언
	public void method3();
}
```

\--> ChildInterface3의 구현객체: method1()과 method2()와 method3()의 실체 메소드 필요
{% endtab %}
{% endtabs %}







{% hint style="info" %}
💡**기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.  애초에 인터페이스를 설계할 때 세심하게 설계하자.**
{% endhint %}



## 어떤 위험이 도사리고 있길래?

디폴트 메소드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 '삽입'될 뿐이다.&#x20;



### **1. 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메소드 작성은 어렵다.**&#x20;



자바8의 Collection 인터페이스에 추가된 removeIf 메소드

{% code title="자바8의 Collection 인터페이스에 추가된 디폴트 메소드" %}
```java
    /**
     * Removes all of the elements of this collection that satisfy the given
     * predicate.  Errors or runtime exceptions thrown during iteration or by
     * the predicate are relayed to the caller.
     *
     * @implSpec
     * The default implementation traverses all elements of the collection using
     * its {@link #iterator}.  Each matching element is removed using
     * {@link Iterator#remove()}.  If the collection's iterator does not
     * support removal then an {@code UnsupportedOperationException} will be
     * thrown on the first matching element.
     *
     * @param filter a predicate which returns {@code true} for elements to be
     *        removed
     * @return {@code true} if any elements were removed
     * @throws NullPointerException if the specified filter is null
     * @throws UnsupportedOperationException if elements cannot be removed
     *         from this collection.  Implementations may throw this exception if a
     *         matching element cannot be removed or if, in general, removal is not
     *         supported.
     * @since 1.8
     */
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```
{% endcode %}

위 코드가 범용적으로 구현된 것처럼 보이지만, 모든 Collection 구현체와 잘 어우러지는 것은 아니다. (ex. `SynchronizedCollection`)

* 이 클래스는 `java.util`의 `Collections.synchronizedCollection` 정적 팩터리 메소드가 반환하는 클래스와 비슷하다. 아파치 버전은 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다. 즉, 모든 메소드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스(item18)이다.&#x20;
* 아파치의 `SynchronizedCollection` 클래스는 지금도 활발히 관리되고 있지만, 이 책을 쓰는 시점엔 removeIf 메소드를 재정의하지 않고 있다. 이 클래스를 자바8과 함께 사용한다면, 자신이 한 약속을 더 이상 지키지 못하게 된다. 다시 말해 모든 메소드 호출을 알아서 동기화해주지 못한다. removeIf의 구현은 동기화에 관해 아무것도 모르므로 락 객체를 사용할 수 없다.  따라서 `SynchronizedCollection 인스턴스`를 여러 스레드가 공유하는 환경에서 한 스레드가 removeIf를 호출하면`ConcurrentModificationException`이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.&#x20;

자바 플랫폼 라이브러리에서도 이런 문제를 예방하기 위해 일련의 조치를 취했다. 예를 들어 인터페이스의 디폴트 메소드를 재정의하고, 다른 메소드에서는 디폴트 메소드를 호출하기 전에 필요한 작업을 수행하도록 했다.&#x20;

예컨대 `Collections.synchronizedCollection`이 반환하는 package-private 클래스들은 removeIf를 재정의하고, 이를 호출하는 다른 메소드들은 디폴트 구현을 호출하기 전에 동기화를 하도록 했다.&#x20;

```java
@Override
public boolean removeIf(Predicate<? super E> filter) {
    synchronized (mutex) {return c.removeIf(filter);}
}
```

하지만 자바 플랫폼에 속하지 않은 제3의 기존 컬렉션 구현체들은 이런 언어 차원의 인터페이스 변화에 발맞춰 수정될 기회가 없었으며, 그 중 일부는 여전히 수정되지 않고 있다.



### 2. 디폴트 메소드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.

자바8은 컬렉션 인터페이스에 꽤 많은 디폴트 메소드들을 추가했고 (4개), 그 결과 기존에 짜여진 많은 자바 코드가 영향을 받은 것으로 알려졌다.&#x20;



## 핵심

<mark style="color:yellow;">**기존 인터페이스에 디폴트 메소드로 새 메소드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.**</mark>&#x20;

추가하려는 디폴트 메소드가 기존 구현체들과 충돌하지는 않을지 심사숙고해야 함도 당연하다. 반면, 새로운 인터페이스를 인터페이스를 만드는 경우라면 표준적인 메소드 구현을 제공하는 데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다. (item 20)



<mark style="color:yellow;">**디폴트 메소드라는 도구가 생겼더라도 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.**</mark>&#x20;

디폴트 메소드로 기존 인터페이스에 새로운 메소드를 추가하면 커다란 위험도 딸려 온다. 인터페이스에 내재된 작은 결함도 사용자 입장에서는 짜증나는 일인데, 심각하게 잘못된 인터페이스라면 이를 포함한 API에 어떤 재앙을 몰고 올지 알 수 없다.

1. 새로운 인터페이스라면 release 전에 반드시 테스트를 거쳐야 한다. 수많은 개발자가 그 인터페이스를 나름의 방식으로 구현할 것이니, 여러분도 서로 다른 방식으로 최소한 세 가지는 구현해봐야 한다.&#x20;
2. 또한 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어봐야 한다.&#x20;

이런 작업들을 거치면 여러분은 인터페이스를 릴리즈하기 전에, 즉 바로잡을 기회가 아직 남았을 때 결함을 찾아낼 수 있다. 인터페이스를 릴리즈한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안 된다.&#x20;

