# item40. @Override 어노테이션을 일관되게 사용하라.

@Override 어노테이션이 달렸다는 것은 상위 타입의 메소드를 재정의했음을 뜻한다.&#x20;

이 어노테이션을 사용하면 여러 가지 악명 높은 버그들을 예방해준다.&#x20;



```java
// Bigram: 영어 알파벳 2개로 구성된 문자열 표현
public class Bigram {
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    public int hashCode() {
        return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i=0; i<10; i++) {
            for (char ch='a'; ch<='z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size()); // 26?
    }
}
```



위 코드는 실제로 260이 출력된다. 무엇이 잘못된 것일까?

<details>

<summary>정답</summary>

Object의 equals를 '재정의(Overriding)'한 것이 아니라 '다중정의(overloading)' 했다.&#x20;

* Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야 한다.

</details>

&#x20;

책에서는 이 오류를 컴파일러가 찾아낼 수 있다고 하는데, 확인해보니 IntelliJ에서 "no usage"로 equals 메소드 위에 "no usage"로 뜨기는 함.

상위 클래스의 메소드를 재정의하려는 모든 메소드에 @Override 어노테이션을 달자. @Override 어노테이션으로 오버라이딩 메소드라는 의미를 명확히 명시할 수 있고, 컴파일러가 해당 재정의에 문제가 있을 시 바로 알려준다.&#x20;



@Override는 클래스 뿐만 아니라 인터페이스의 메소드를 재정의할 때도 사용한다.  자바8부터 디폴트 메소드를 지원하기 시작하면서, 인터페이스 메소드를 구현한 메소드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다.&#x20;

* 구현하려는 인터페이스에 디폴트 메소드가 없음을 안다면 구현클래스에서는 모든 메소드에 @Override를 빼는 것도 깔끔해지기는 함.

BUT, 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메소드를 재정의하는 모든 메소드에 @Override를 다는 것이 좋다. 상위 클래스가 구체 클래스든 추상 클래스든 마찬가지!

* ex) Set 인터페이스는 Collection 인터페이스를 확장했지만 새로 추가된 메소드는 없다. 따라서 모든 메소드 선언에 @Override를 달아 실수로 추가한 메소드가 없음을 보장했다.&#x20;



{% hint style="info" %}
💡 재정의한 모든 메소드에 @Override 어노테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다.&#x20;

예외는 단 한가지뿐, 쿠체 클래스에서 상위 클래스의 추상 메소드를 재정의한 경우엔 이 어노테이션을 달지 않아도 된다.  (단다고 해서 해로울 건 없다)
{% endhint %}



