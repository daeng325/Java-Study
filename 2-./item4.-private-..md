# item4. 인스턴스화를 막으려거든 private 생성자를 사용하라.

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다.

하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자인 매개변수를 받지 않는 public 생성자를 만들어준다. (사용자는 이게 컴파일러가 자동 생성한 건지 의도적으로 인스턴스화할 수 있게 만든 것인지 구별하지 못한다.)

해당 클래스들의 인스턴스화를 막으려면 어떻게 해야 하는가?

{% hint style="info" %}
💡 **private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**
{% endhint %}

* 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. 하위 클래스를 만들어서 인스턴스화하면 끝.
  * 그리고 추상클래스로 만들어버리면 사용자는 이걸 상속해서 사용하라고 만들어진 거구나 라고 오해할 수 있으니 더 큰 문제가 됨

```jsx
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다. (인스턴스화 방지용)
	private UtilityClass() {
		throw new AssersionError();
	}
}
```

1. 명시적 생성자가 private 이니 클래스 바깥에서는 접근할 수 없음
2. AssertionError를 던지게 되면, 클래스 안에서 실수로 생성자를 호출했을 경우를 방지
3. 생성자가 존재하는데 호출할수는 없다는 것이 직관적이지 않으니 적절한 주석을 달아주기
4.  이 방식은 상속을 불가능하게 하는 효과도 존재

    → 모든 생성자는 명시적이든 묵시적이든 상위 클래스 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스 생성자에 접근할 길이 막혀버린다.

    [https://stackoverflow.com/questions/38945400/why-am-i-able-to-inherit-call-a-private-constructor-in-a-subclass](https://stackoverflow.com/questions/38945400/why-am-i-able-to-inherit-call-a-private-constructor-in-a-subclass)

    [https://stackoverflow.com/questions/16661595/why-can-you-not-inherit-from-a-class-whose-constructor-is-private](https://stackoverflow.com/questions/16661595/why-can-you-not-inherit-from-a-class-whose-constructor-is-private)
