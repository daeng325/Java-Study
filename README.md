---
cover: >-
  https://images.unsplash.com/photo-1616400619175-5beda3a17896?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw4fHxzdHVkeXxlbnwwfHx8fDE2NzkxODc3NjQ&ixlib=rb-4.0.3&q=80
coverY: 0
---

# 🍀 Effective Java #3

<img src=".gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

## 2장: 객체 생성과 파괴

### item4. 인스턴스화를 막으려거든 private 생성자를 사용하라.

<details>

<summary>item4. 인스턴스화를 막으려거든 private 생성자를 사용하라.</summary>

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다.

하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자인 매개변수를 받지 않는 public 생성자를 만들어준다. (사용자는 이게 컴파일러가 자동 생성한 건지 의도적으로 인스턴스화할 수 있게 만든 것인지 구별하지 못한다.)

해당 클래스들의 인스턴스화를 막으려면 어떻게 해야 하는가?

💡 \*\*private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.\*\*

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

</details>



## 3장: 모든 객체의 공통 메소드

### item11. equals를 재정의하려거든 hashCode도 재정의하라.

<details>

<summary>item11. equals를 재정의하려거든 hashCode도 재정의하라.</summary>

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

equals만 재정의하면 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

Object 명세서

* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없다.
* equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
*   equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

    [\[자료구조\] 해시테이블(HashTable)이란?](https://mangkyu.tistory.com/102)

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

Example Code

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309)); // null을 return
```

위 예제에서 PhoneNumber가 hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.

* get 메소드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 할 것
* 두 인스턴스를 같은 버킷에 담았다고 하더라도, get 메소드는 여전히 null 반환
  * Why? HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화되어 있음.

#### 올바른 hashCode 메소드는 어떤 모습이어야 할까?

사용 금지!!!

```java
@Override public int hashCode() { return 42;}
```

개끔찍. 모든 객체에서 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 linked list처럼 동작. —> 평균 수행 시간이 O(1)인 해시테이블이 O(N)으로 느려진다.

좋은 hash function? 서로 다른 인스턴스에 다른 해시코드를 반환하는 것. —> hashCode의 세 번째 규약이 요구하는 속성.

이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32bit 정수 범위에 균일하게 분배해야 한다. 이상을 완벽히 실현하기는 어렵지만 비슷하게 만들기는 그다지 어렵지 않다.

**hashCode를 작성하는 간단한 요령**

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이 때 c는 해당 객체의 첫 번째 핵심 필드(equals 비교에 사용되는 필드를 뜻함)를 단계 2.a 방식으로 계산한 해시코드다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 c를 계산한다.
      1. primitive type이라면, Type.hashCode(f)를 수행한다. \*\*Type ⇒ Boxing Class
      2.  reference type 필드이면서 이 클래스의 equals 메소드가 이 필드의 equals를 recursive로 호출해 비교한다면, 이 필드의 hashCode를 recursive로 호출한다. 계산이 복잡해질 것 같으면, 이 필드의 canonical representation(표준형)을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. (다른 상수도 괜찮지만 전통적으로 0을 사용한다. )

          ~~진짜 뭔말이야… canonical… 뭐? 재귀적으로.. 뭘 해?~~
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
   2.  단계 2.a 에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.

       ```java
       result = 31 * result + c; 
       // 31 * result: 필드를 곱하는 순서에 따라 result값이 달라지게 한다. 
       // 31인 이유? 홀수이면서 prime이기 때문. 이 숫자가 짝수이고 오버플로우가 발생한다면 정보를 잃게 된다.
       // (31*i = (i<<5) -i) shift 연산 및 뺄셈으로 곱셈을 최적화할 수 있다.
       ```
   3. result를 반환한다.

```java
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

Hash Collision이 더 적은 방법을 꼭 써야 한다면 구아바의 com.google.common.hash.Hashing 참고

[Hashing (Guava: Google Core Libraries for Java 21.0 API)](https://guava.dev/releases/21.0/api/docs/com/google/common/hash/Hashing.html)

```java
@Override public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

* Objects.hash(Objects… values) 의 parameter를 담기 위한 배열이 만들어지고, 입력 중 primitive type이 있다면 Boxing 및 UnBoxing을 거쳐야 하기 때문에 느려지는 것
* 따라서 해당 메소드는 성능에 민감하지 않은 상황에서만 사용하자. (제공부는 안되겠고 수집부 같은 곳에서는 훨씬 가독성 있어 보임)

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 그리고 또 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해두는 방법도 좋음 (캐싱 방식)

해시코드를 계산하는 비용이 큰데 해시의 키로 사용되지 않는 경우라면 hashCode()가 처음 불릴 때 계산하는 지연 초기화(lazy initialization) 전략도 좋음 (필드 지연 초기화 하려면 thread-safe해야 함. item83)

```java
private int hashCode; // 0으로 초기화

@Override public int hashCode() {
	int result = hashCode;
	if (result ==0) { 
		int result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
```

* **item83. 지연 초기화는 신중히 사용하라.**
  * 멀티스레드 환경에서는 lazy initialization을 하기가 까다롭다. 해당 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.
  *   성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중 검사(double-check) 관용구를 사용하라. 한 번은 동기화 없이 검사하고, (필드가 아직 초기화되지 않았다면) 두 번째는 동기화하여 검사한다. 두 번째 검사에서도 필드가 초기화되지 않았을 때만 필드를 초기화한다. 초기화된 이후로는 동기화하지 않으므로 해당 필드는 반드시 `volatile`로 선언해야 한다.

      ```java
      private volatile FieldType field;

      private FieldType getField() {
      	FieldType result = field; // result 지역변수 필요한 이유: 필드가 이미 초기화된 상황에서 그 필드를 딱 한 번만 읽도록 보장하는 역할.
      	if (result != null) { // 첫 번째 검사 (lock 사용 안 함)
      		return result;
      	}
      	synchronized(this) {
      		if (field == null) { // 두 번째 검사 (lock 사용)
      			field = computeFieldValue();
      		return field;
      	}
      }
      ```

[Pull Request #19: \[feat\] add the logic for providing the data according to request based on poi and radius - Bitbucket (ccos.dev)](https://repo.ccos.dev/projects/MCP1/repos/kr-mcp-interface/pull-requests/19/commits/9300e9a602ab98163bd3a19b23f279e4944824c6#src/main/java/mcpi/application/ev/service/binary/entity/EvStationInfo.java)

</details>



## 4장. 클래스와 인터페이스

### **item15. 클래스와 멤버의 접근 권한을 최소화하라.**

<details>

<summary>item15. 클래스와 멤버의 접근 권한을 최소화하라.</summary>

잘 설계된 컴포넌트: 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐, 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다. 오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다.

정보은닉, 혹은 캡슐화!

정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다. 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.

💡 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심이다. \*\*모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.\*\*

멤버 (필드, 메소드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준은 네 가지다.

**private**: 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.

**package-private** (default): 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다. 접근제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준

**protected**: package-private 의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다. ~~(제약이 조금 따른다는데 뭔지는 알려줘야지..)~~

**public**: 모든 곳에서 접근 가능

모든 클래스와 멤버의 접근성을 가능한 한 좁히자.

* 톱레벨 클래스나 인터페이스를 패키지 외부에서 쓸 이유가 없다면 package-private으로 선언하자. (default) , public으로 선언하면 API가 되므로 하위 호환을 위해 영원히 관리해줘야 한다.
* 한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시켜보자. (item24) 톱레벨로 두면 같은 패키지의 모든 클래스가 접근할 수 있지만, private static이면 바깥 클래스 하나에서만 접근할 수 있다.
  * 그냥 private으로만 하면 되는 거 아니야? 굳이 static을 붙여야 하나?
  *   **item24. 멤버 클래스는 되도록 static으로 만들라.**

      static 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다. 그래서 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 static 멤버 클래스로 만들어야 한다.

      **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.** static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다. **참조를 저장하려면 시간과 공간이 소비된다. 더 큰 문제는 GC가 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 더 문제점이다**. (item 7)

      private static 멤버클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.

      * 많은 Map 구현체는 Entry 객체들을 가지고 있다. 모든 Entry가 map과 연관되어 있지만 Entry의 method들은 map을 직접 사용하지는 않는다. 그래서 인스턴스 멤버 클래스로 표현하는 건 낭비고, private static 멤버 클래스가 가장 알맞다.
      * 물론 static을 빠뜨려도 Map은 여전히 동작하지만, 모든 entry가 바깥 map으로의 참조를 갖게 되어 공간과 시간을 낭비할 것이다.

      ```java
      public class A{
      	int field1;
      	void method1(){}
      	
      	static int field2;
      	static void method2(){}
      	
      	class B{
      		void method(){
      			field1=10;
      			method1();
      			
      			field2=10;
      			method2();
      		}
      	}
      	
      	static class C{
      		void method(){
      			// field1=10;
      			// method1();
      			
      			field2=10;
      			method2();
      		}
      	}
      }

      A.C c = new A.C();
      c.field1=3; // 인스턴스 필드 사용
      c.method1(); // 인스턴스 메소드 호출
      A.C.field2=3; // 정적 필드 사용
      A.C.method2(); // 정적 메소드 호출
      ```

클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만들고, 그 다음 오직 같은 패키지의 다른 클래스가 접근해야 할 일이 생기면 그제서야 제한자를 package-private으로 풀어주자. 권한을 풀어주는 일이 너무 잦다면 컴포넌트로 따로 빼야 되는 것이 아닌지 다시 고민해 보자.

* 단, Serializable을 구현한 클래스에서는 그 필드들도 의도치 않게 공개 API가 될 수 있다. (item 86, 87)

멤버 접근성을 좁히지 못하게 방해하는 제약이 하나 있다. 상위 클래스의 메소드를 오버라이딩할때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다.

* 리스코프 치환 원칙 (item 10) : 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.
* 클래스가 인터페이스를 구현하는 건 이 규칙의 특별한 예로 볼 수 있고, 이 때 클래스는 인터페이스가 정의한 모든 메소드를 public으로 선언해야 한다.

**public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다**. (item16) 필드가 가변 객체를 참조하거나, final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃게 된다.

여기에 더해, 필드가 수정될 때 (락 획득 같은) 다른 작업을 할 수 없게 되므로 _**public 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다.**_ **심지어 필드가 final이면서 불변 객체를 참조하더라도 문제는 여전히 남는다**. ~~????????????~~ 내부 구현을 바꾸고 싶어도 그 public 필드를 없애는 방식으로는 리팩토링할 수 없게 된다. public final 안변하는객체; —> 얘를 사용하는 모든 곳에서 다 고쳐야 된다?

길이가 0이 아닌 배열은 모두 변경 가능하니 주의하자. 따라서 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메소드를 제공해서는 안 된다. 이런 필드나 접근자를 제공한다면 클라이언트에서 그 배열의 내용을 수정할 수 있게 된다.

1. public 배열을 private으로 만들고 public 불변 리스트를 추가. `(Collections.unmodifiableList(Arrays.asList(ARR));`
2. **배열을 private으로 만들고 그 복사본을 반환하는 public 메소드 추가**. `ARR.clone();`
   1. ~~근데 이러면 얕은 복사 아닌가? ARR이 primitive type이 아니라 객체 타입이면 어쩔 건데? 이건 IntelliJ에서 보기… Clone 메소드가 오버라이딩이 되어 있나?~~

* public 클래스에서 멤버 접근 수준을 package-private에서 protected로 바뀌는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청나게 넓어진다. public 클래스의 protected 멤버는 공개 API이므로 영원히 지원돼야 한다. protected 멤버 수는 적을수록 좋다.

<!---->

* 단, 인터페이스의 멤버는 기본적으로 public이 적용된다. (인터페이스란 객체의 사용방법을 알려주는 명세서 같은 거니까 그런 듯?)

</details>

### item16. public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라.

<details>

<summary>item16. public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라.</summary>

```java
class Point{
	public double x;
	public double y;
}
```

* **API의 필드를 수정하지 않고는 내부 표현을 바꿀 수 없다.**
  * getter/setter 메서드가 존재한다면 얼마든지 표현 변경이 가능하다.
* **불변식을 보장할 수 없다.**
  * 클라이언트가 직접 데이터를 변경할 수 있다.
* **외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.**
  * 1차원적인 접근만 가능하고, 추가 로직을 삽입할 수 없다.

철저한 객체 지향 프로그래머는 필드를 모두 private으로 바꾸고 public getter를 추가한다.

```java
class Point {
        private double x;
        private double y;

        public Point(double x, double y) {
            this.x = x;
            this.y = y;
        }

        public double getX() {
            return x;
        }

        public void setX(double x) {
            this.x = x;
        }

        public double getY() {
            return y;
        }

        public void setY(double y) {
            this.y = y;
        }
    }
```

public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

—> 아 이게 사용하는 입장에서 얘기하는 게 아니라, 클래스를 제공한 입장에서의 얘기구나.

package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. (접근자 방법보다 훨씬 깔끔하기도 함)

* **package-private 클래스에서 public 데이터 필드 노출**: 클라이언트 코드가 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.
* **private 중첩 클래스에서 public 데이터 필드 노출**: 이 경우는 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다. 대표적인 예로 `java.awt` 패키지의 `Point`와 `Dimension`클래스이다.

* item67에서 설명하듯, 내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘까지도 해결되지 못했다.
*   참고 (Point , Dimension 클래스)

    ```java

    package java.awt;

    import java.awt.geom.Point2D;
    import java.beans.Transient;

    public class Point extends Point2D implements java.io.Serializable {
        /**
         * The X coordinate of this <code>Point</code>.
         * If no X coordinate is set it will default to 0.
         *
         * @serial
         * @see #getLocation()
         * @see #move(int, int)
         * @since 1.0
         */
        public int x;

        /**
         * The Y coordinate of this <code>Point</code>.
         * If no Y coordinate is set it will default to 0.
         *
         * @serial
         * @see #getLocation()
         * @see #move(int, int)
         * @since 1.0
         */
        public int y;

        /*
         * JDK 1.1 serialVersionUID
         */
        private static final long serialVersionUID = -5276940640259749850L;

        /**
         * Constructs and initializes a point at the origin
         * (0,&nbsp;0) of the coordinate space.
         * @since       1.1
         */
        public Point() {
            this(0, 0);
        }

        /**
         * Constructs and initializes a point with the same location as
         * the specified <code>Point</code> object.
         * @param       p a point
         * @since       1.1
         */
        public Point(Point p) {
            this(p.x, p.y);
        }

        /**
         * Constructs and initializes a point at the specified
         * {@code (x,y)} location in the coordinate space.
         * @param x the X coordinate of the newly constructed <code>Point</code>
         * @param y the Y coordinate of the newly constructed <code>Point</code>
         * @since 1.0
         */
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getX() {
            return x;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getY() {
            return y;
        }

        /**
         * Returns the location of this point.
         * This method is included for completeness, to parallel the
         * <code>getLocation</code> method of <code>Component</code>.
         * @return      a copy of this point, at the same location
         * @see         java.awt.Component#getLocation
         * @see         java.awt.Point#setLocation(java.awt.Point)
         * @see         java.awt.Point#setLocation(int, int)
         * @since       1.1
         */
        @Transient
        public Point getLocation() {
            return new Point(x, y);
        }

        /**
         * Sets the location of the point to the specified location.
         * This method is included for completeness, to parallel the
         * <code>setLocation</code> method of <code>Component</code>.
         * @param       p  a point, the new location for this point
         * @see         java.awt.Component#setLocation(java.awt.Point)
         * @see         java.awt.Point#getLocation
         * @since       1.1
         */
        public void setLocation(Point p) {
            setLocation(p.x, p.y);
        }

        /**
         * Changes the point to have the specified location.
         * <p>
         * This method is included for completeness, to parallel the
         * <code>setLocation</code> method of <code>Component</code>.
         * Its behavior is identical with <code>move(int,&nbsp;int)</code>.
         * @param       x the X coordinate of the new location
         * @param       y the Y coordinate of the new location
         * @see         java.awt.Component#setLocation(int, int)
         * @see         java.awt.Point#getLocation
         * @see         java.awt.Point#move(int, int)
         * @since       1.1
         */
        public void setLocation(int x, int y) {
            move(x, y);
        }

        /**
         * Sets the location of this point to the specified double coordinates.
         * The double values will be rounded to integer values.
         * Any number smaller than <code>Integer.MIN_VALUE</code>
         * will be reset to <code>MIN_VALUE</code>, and any number
         * larger than <code>Integer.MAX_VALUE</code> will be
         * reset to <code>MAX_VALUE</code>.
         *
         * @param x the X coordinate of the new location
         * @param y the Y coordinate of the new location
         * @see #getLocation
         */
        public void setLocation(double x, double y) {
            this.x = (int) Math.floor(x+0.5);
            this.y = (int) Math.floor(y+0.5);
        }

        /**
         * Moves this point to the specified location in the
         * {@code (x,y)} coordinate plane. This method
         * is identical with <code>setLocation(int,&nbsp;int)</code>.
         * @param       x the X coordinate of the new location
         * @param       y the Y coordinate of the new location
         * @see         java.awt.Component#setLocation(int, int)
         */
        public void move(int x, int y) {
            this.x = x;
            this.y = y;
        }

        /**
         * Translates this point, at location {@code (x,y)},
         * by {@code dx} along the {@code x} axis and {@code dy}
         * along the {@code y} axis so that it now represents the point
         * {@code (x+dx,y+dy)}.
         *
         * @param       dx   the distance to move this point
         *                            along the X axis
         * @param       dy    the distance to move this point
         *                            along the Y axis
         */
        public void translate(int dx, int dy) {
            this.x += dx;
            this.y += dy;
        }

        /**
         * Determines whether or not two points are equal. Two instances of
         * <code>Point2D</code> are equal if the values of their
         * <code>x</code> and <code>y</code> member fields, representing
         * their position in the coordinate space, are the same.
         * @param obj an object to be compared with this <code>Point2D</code>
         * @return <code>true</code> if the object to be compared is
         *         an instance of <code>Point2D</code> and has
         *         the same values; <code>false</code> otherwise.
         */
        public boolean equals(Object obj) {
            if (obj instanceof Point) {
                Point pt = (Point)obj;
                return (x == pt.x) && (y == pt.y);
            }
            return super.equals(obj);
        }

        /**
         * Returns a string representation of this point and its location
         * in the {@code (x,y)} coordinate space. This method is
         * intended to be used only for debugging purposes, and the content
         * and format of the returned string may vary between implementations.
         * The returned string may be empty but may not be <code>null</code>.
         *
         * @return  a string representation of this point
         */
        public String toString() {
            return getClass().getName() + "[x=" + x + ",y=" + y + "]";
        }
    }
    ```

    ```java
    package java.awt;

    import java.awt.geom.Dimension2D;
    import java.beans.Transient;

    public class Dimension extends Dimension2D implements java.io.Serializable {

        /**
         * The width dimension; negative values can be used.
         *
         * @serial
         * @see #getSize
         * @see #setSize
         * @since 1.0
         */
        public int width;

        /**
         * The height dimension; negative values can be used.
         *
         * @serial
         * @see #getSize
         * @see #setSize
         * @since 1.0
         */
        public int height;

        /*
         * JDK 1.1 serialVersionUID
         */
         private static final long serialVersionUID = 4723952579491349524L;

        /**
         * Initialize JNI field and method IDs
         */
        private static native void initIDs();

        static {
            /* ensure that the necessary native libraries are loaded */
            Toolkit.loadLibraries();
            if (!GraphicsEnvironment.isHeadless()) {
                initIDs();
            }
        }

        /**
         * Creates an instance of <code>Dimension</code> with a width
         * of zero and a height of zero.
         */
        public Dimension() {
            this(0, 0);
        }

        /**
         * Creates an instance of <code>Dimension</code> whose width
         * and height are the same as for the specified dimension.
         *
         * @param    d   the specified dimension for the
         *               <code>width</code> and
         *               <code>height</code> values
         */
        public Dimension(Dimension d) {
            this(d.width, d.height);
        }

        /**
         * Constructs a <code>Dimension</code> and initializes
         * it to the specified width and specified height.
         *
         * @param width the specified width
         * @param height the specified height
         */
        public Dimension(int width, int height) {
            this.width = width;
            this.height = height;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getWidth() {
            return width;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getHeight() {
            return height;
        }

        /**
         * Sets the size of this <code>Dimension</code> object to
         * the specified width and height in double precision.
         * Note that if <code>width</code> or <code>height</code>
         * are larger than <code>Integer.MAX_VALUE</code>, they will
         * be reset to <code>Integer.MAX_VALUE</code>.
         *
         * @param width  the new width for the <code>Dimension</code> object
         * @param height the new height for the <code>Dimension</code> object
         * @since 1.2
         */
        public void setSize(double width, double height) {
            this.width = (int) Math.ceil(width);
            this.height = (int) Math.ceil(height);
        }

        /**
         * Gets the size of this <code>Dimension</code> object.
         * This method is included for completeness, to parallel the
         * <code>getSize</code> method defined by <code>Component</code>.
         *
         * @return   the size of this dimension, a new instance of
         *           <code>Dimension</code> with the same width and height
         * @see      java.awt.Dimension#setSize
         * @see      java.awt.Component#getSize
         * @since    1.1
         */
        @Transient
        public Dimension getSize() {
            return new Dimension(width, height);
        }

        /**
         * Sets the size of this <code>Dimension</code> object to the specified size.
         * This method is included for completeness, to parallel the
         * <code>setSize</code> method defined by <code>Component</code>.
         * @param    d  the new size for this <code>Dimension</code> object
         * @see      java.awt.Dimension#getSize
         * @see      java.awt.Component#setSize
         * @since    1.1
         */
        public void setSize(Dimension d) {
            setSize(d.width, d.height);
        }

        /**
         * Sets the size of this <code>Dimension</code> object
         * to the specified width and height.
         * This method is included for completeness, to parallel the
         * <code>setSize</code> method defined by <code>Component</code>.
         *
         * @param    width   the new width for this <code>Dimension</code> object
         * @param    height  the new height for this <code>Dimension</code> object
         * @see      java.awt.Dimension#getSize
         * @see      java.awt.Component#setSize
         * @since    1.1
         */
        public void setSize(int width, int height) {
            this.width = width;
            this.height = height;
        }

        /**
         * Checks whether two dimension objects have equal values.
         */
        public boolean equals(Object obj) {
            if (obj instanceof Dimension) {
                Dimension d = (Dimension)obj;
                return (width == d.width) && (height == d.height);
            }
            return false;
        }

        /**
         * Returns the hash code for this <code>Dimension</code>.
         *
         * @return    a hash code for this <code>Dimension</code>
         */
        public int hashCode() {
            int sum = width + height;
            return sum * (sum + 1)/2 + width;
        }

        /**
         * Returns a string representation of the values of this
         * <code>Dimension</code> object's <code>height</code> and
         * <code>width</code> fields. This method is intended to be used only
         * for debugging purposes, and the content and format of the returned
         * string may vary between implementations. The returned string may be
         * empty but may not be <code>null</code>.
         *
         * @return  a string representation of this <code>Dimension</code>
         *          object
         */
        public String toString() {
            return getClass().getName() + "[width=" + width + ",height=" + height + "]";
        }
    }
    ```

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 단, 불변식은 보장할 수 있게 된다.

```java
public class Time {
  private static final int HOURS_PER_DAY = 24;
  private static final int MINUTES_PER_HOUR = 60;
  
  public final int hour;
  public final int minute;
  
  public Time (int hour, int minute) { // 각 인스턴스가 유효한 시간을 표현함을 보장.
      if (hour < 0 || hour >= HOURS_PER_DAY) 
          throw new IllegalArgumentException("시간: " + hour);
      if (minute < 0 || minute >= MINUTES_PER_HOUR)
          throw new IllegalArgumentException("분: " + minute);
      this.hour = hour;
      this.minute = minute;
  }
	// 나머지 코드 생략
}
```

<mark style="background-color:yellow;">💡 \*\*public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서은 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.\*\*</mark>

</details>



## 9장: 일반적인 프로그래밍 원칙

### item65. Reflection보다는 Interface를 사용하라.

<details>

<summary>item65. Reflection보다는 Interface를 사용하라.</summary>

Reflection이란? 런타임 시에 클래스의 메타 정보를 얻는 기능을 말한다.

클래스가 가지고 있는 필드가 무엇인지, 어떤 생성자를 갖고 있는지, 어떤 메소드를 갖고 있는지, 적용된 어노테이션이 무엇인지 알아내는 것이 리플렉션이다.

java.lang.reflect 의 리플렉션 기능을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.

Class 객체로 가져올 수 있는 인스턴스

* Constructor
* Method
* Field

위 인스턴스들로는 그 클래스의 멤버 이름, 필드 타입, 메소드 시그니처 등을 가져올 수 있다.

또한, 이 인스턴스들을 이용해 각각에 연결된 실제 생성자, 메소드, 필드를 조작할 수도 있다. 이 인스턴스들을 통해 해당 클래스의 인스턴스를 생성하거나, 메소드를 호출하거나, 필드에 접근할 수 있다는 뜻이다.

리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.

#### Reflection의 단점

1. **컴파일타임 type 검사가 주는 이점을 하나도 누릴 수 없다.** (예외 검사도 마찬가지)
   * 프로그램이 리플렉션 기능을 써서 존재하지 않는 혹은 접근할 수 없는 메소드를 호출하려 시도하면 런타임 오류가 발생한다. (주의해서 대비 코드를 작성해 두어야 함)
2. **코드가 지저분하고 장황해진다.**
3. **성능이 떨어진다.**
   * 리플렉션을 통한 메소드 호출은 일반 메소드 호출보다 훨씬 느리다. (고려해야하는 요소가 많아 정확한 차이는 이야기하기 어려움). 입력 매개변수가 없고 int를 반환하는 메소드로 실험해 보니 11배나 느렸다고 함.

**리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다.**

컴파일 타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을 것이다. (아이템 64)

**리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.**

```java

public static void main(String[] args) {
    // argument로 들어온 클래스 이름을 통해 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        throw new RuntimeException("클래스를 찾을 수 없습니다." , e);
    }

    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor(); // 해당 클래스에 선언된 매개변수 없는 생성자를 가져옴
    } catch (NoSuchMethodException e) {
        throw new RuntimeException("매개변수 없는 생성자를 찾을 수 없습니다.", e);
    }

    // Set 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (InvocationTargetException e) {
        throw new RuntimeException("생성자가 예외를 던졌습니다.", e);
    } catch (InstantiationException e) {
        throw new RuntimeException("클래스를 인스턴스화할 수 없습니다.", e);
    } catch (IllegalAccessException e) {
        throw new RuntimeException("생성자에 접근할 수 없습니다.", e);
    } catch (ClassCastException e) {
        throw new RuntimeException("Set을 구현하지 않은 클래스입니다.", e);
    }

    // 생성한 Set을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}
```

위 예시는 2가지의 단점을 보인다.

1.  런타임에 총 6가지나 되는 예외를 던질 수 있다.



    그 모두가 인스턴스를 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었을 예외들이다.

    \-→ InvocationTargetException은 아무리 해도 안 나오는데..? 공책임님 말로는 생성자가 public이지만 바로 throw를 해버려서 그 생성자를 못 쓰게 만든 경우도 있다고 함. 그럴 경우에는 저 Exception이 날 것 같음.

    1.  &#x20;args: Set



        <figure><img src=".gitbook/assets/Untitled.png" alt=""><figcaption></figcaption></figure>
    2.  args: java.util.Set (NoSuchMethodException)\


        <figure><img src=".gitbook/assets/Untitled 1.png" alt=""><figcaption></figcaption></figure>
    3.  args: jata.util.EnumSet\
        (InstantiationException: 해당 클래스가 추상클래스이거나 인터페이스인 경우)



        <figure><img src=".gitbook/assets/Untitled 2.png" alt=""><figcaption></figcaption></figure>
    4.  args: java.util.AbstractSet

        ```java
        Exception in thread "main" java.lang.RuntimeException: 생성자에 접근할 수 없습니다.
        	at com.company.example.EffectiveJava.item65(EffectiveJava.java:40)
        	at com.company.example.EffectiveJava.main(EffectiveJava.java:11)
        Caused by: java.lang.IllegalAccessException: Class com.company.example.EffectiveJava can not access a member of class java.util.AbstractSet with modifiers "protected"
        	at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:102)
        	at java.lang.reflect.AccessibleObject.slowCheckMemberAccess(AccessibleObject.java:296)
        	at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:288)
        	at java.lang.reflect.Constructor.newInstance(Constructor.java:413)
        	at com.company.example.EffectiveJava.item65(EffectiveJava.java:34)
        	... 1 more
        ```


    5.  args: java.util.ArrayList\


        <figure><img src=".gitbook/assets/Untitled 3.png" alt=""><figcaption></figcaption></figure>
2. 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 작성했다. 리플렉션이 아니라면 생성자 호출 한 줄로 끝났을 일이다.
   * 참고로, 리플렉션 예외 각각을 잡는 대신 모든 리플렉션 예외의 상위 클래스인 `ReflectiveOperationException`을 잡도록 하여 코드 길이를 줄일 수는 있다.

두 단점 모두 객체를 생성하는 부분에만 국한된다. 객체가 일단 만들어지면 그 후의 코드는 여타의 Set 인스턴스를 사용할 때와 똑같다. 그래서 실제 프로그램에서는 이런 제약에 영향 받는 코드는 일부에 지나지 않는다.

이 프로그램을 컴파일하면 Unchekced Cast 경고가 뜬다.

*

    <figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

하지만 `Class<? extends Set<String>>` 으로의 형변환은 심지어 명시한 클래스가 Set을 구현하지 않았더라도 성공한다. 단, 그 클래스의 인스턴스를 생성하려 할 때 `ClassCastException`을 던지게 된다. 이 경고를 숨기는 방법은 아이템 27 참고.

* **item27. @SuppressWarnings(”unchecked”) 어노테이션을 사용해 경고 숨기기**
  * 단, 타입 안전성을 확신할 수 있을 경우에만 해당 어노테이션 사용 필요
  * 해당 어노테이션을 사용했다면 반드시 그 경고를 무시해도 안전한 이유를 주석으로 남겨야 한다. 다른 사람이 그 코드를 이해하는데 도움이 되며, 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여주기 때문이다.

드물긴 하지만, 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메소드, 필드와의 의존성을 관리할 때 적합하다.

이 기법은 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용하다. 가동할 수 있는 최소한의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일한 후, 이후 버전의 클래스와 메소드 등은 리플렉션으로 접근하는 방식이다.

이렇게 하려면 접근하려는 새로운 클래스나 메소드가 런타임에 존재하지 않을 수 있다는 사실을 반드시 감안해야 한다. 즉, 같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄여 동작하는 등의 적절한 조치를 취해야 한다.

핵심 정리

* 리플렉션은 복잡한 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다. 컴파일타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성한다면 리플렉션을 사용해야 할 것이다. 단, 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다.

aspectj 예시 보여주기

</details>



### item67. 최적화는 신중히 하라.

<details>

<summary>item67. 최적화는 신중히 하라.</summary>

최적화는 좋은 결과보다는 해로운 결과로 이어지기 쉽고, 섣불리 진행하면 특히 더 그렇다. 빠르지도 않고 제대로 동작하지도 않으면서 수정하기는 어려운 소프트웨어를 탄생시킬 것인가?

성능 때문에 견고한 구조를 희생하지 말자. **빠른 프로그램보다는 좋은 프로그램을 작성하라.**

좋은 프로그램이지만 원하는 성능이 나오지 않는다면?—> 그 아키텍처 자체가 최적화할 수 있는 길을 안내해줄 것

좋은 프로그램은 정보 은닉 원칙을 따르므로 개별 구성요소의 내부를 독립적으로 설계할 수 있다. 따라서 시스템의 나머지에 영향을 주지 않고도 각 요소를 다시 설계할 수 있다. (item15)

💡 \*\*설계 단계에서 성능을 반드시 염두에 두어야 한다.\*\*

1. **성능을 제한하는 설계를 피하라.**
   1.  컴포넌트끼리, 혹은 외부 시스템과의 소통방식 관련한 설계 요소(API, 네트워크 프로토콜, 영구 저장용 데이터 포맷 등)는 완성 후 가장 변경하기가 어렵다.

       —> 위 설계 요소들이 잘못되면 성능을 심각하게 제한할 수 있다.
2. **API를 설계할 때 성능에 주는 영향을 고려하라.**
   1. public 타입을 가변으로 만들면 불필요한 방어적 복사를 수없이 유발할 수 있다. (item50)
      *   **item 50. 적시에 방어적 복사본을 만들라.**

          메소드이든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다. 변경될 수 있는 객체라면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제없이 동작할지를 따져보라. 확신할 수 없다면 복사본을 만들어 저장해야 한다.

          ex) 클라이언트가 건네준 객체를 내부의 Set인스턴스에 저장하거나, Map 인스턴스의 키로 사용한다면? 추후 그 객체가 변경될 경우 객체를 담고 있는 Set 혹은 Map의 불변식이 깨질 것이다.

          내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만드는 이유도 마찬가지다. 내 클래스가 불변이든 가변이든, 그 안의 내부 객체가 가변이면 클라이언트에 반환할 때 반드시 심사숙고해야 한다. 안심할 수 없다면 방어적 복사본을 반환해야 한다.

          길이가 1 이상인 배열은 무조건 가변임을 잊지 말자. 그러니 내부에서 사용하는 배열은 클라이언트에 반환할 때는 항상 방어적 복사를 수행해야 한다. 혹은 배열의 불변 뷰를 반환하는 대안도 있다. **(item 15)**

          통제권을 넘겨받기로 한 메소드나 생성자를 가진 클래스들은 악의적인 클라이언트의 공격에 취약하다. 따라서 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야 한다. 후자의 예로는 래퍼 클래스 패턴(item18)을 들 수 있다. 래퍼 클래스 특성 상 클라이언트는 래퍼에 넘긴 객체에 여전히 접근할 수 있다. 따라서 래퍼의 불변식을 쉽게 파괴할 수 있지만 그 영향을 오직 클라이언트 자신만 받게 된다.
      * **그래서 우리의 Dto나 VO들은 다 잘못 설계되어 있는 걸까?**
   2. 컴포지션으로 해결할 수 있음에도 상속 방식으로 설계한 public 클래스는 상위 클래스에 영원히 종속되며 그 성능 제약까지도 물려받게 된다. (item18)
   3. 인터페이스도 있는데 굳이 구현 타입을 사용하는 것은 특정 구현체에 종속되게 한다. (item64)
3. **그렇다고 성능을 위해 API를 왜곡하지는 말라.**
   1. 잘 설계된 API는 성능도 좋은 게 보통이며, 성능 때문에 API를 왜곡하도록 만든 성능 문제는 해당 플랫폼이나 아랫단 소프트웨어가 버전업되면 사라질 수 있지만, 왜곡된 API로 인해 영원히 고통받을 것이다. ~~API를 왜곡한다는 게 대체 뭔 말인지 모르겠다만…~~

신중하게 설계하여 좋은 구조를 갖춘 프로그램을 완성한 다음에야 최적화를 고려해볼 차례가 된다!

그래서 최적화는?

1. 하지 마라.
2. (전문가 한정) 아직 하지 마라.
3. **각각의 최적화 시도 전후로 성능을 측정하라.**
   1. 자바의 경우 프로그래머가 작성하는 코드와 CPU에서 수행하는 명령 사이의 ‘추상화 격차’가 커서 최적화로 인한 성능 변화를 일정하게 예측하기가 그만큼 더 어렵다.
   2. 자바의 성능 모델은 정교하지 않고 구현 시스템, 릴리즈, 프로세서마다의 차이가 있고, 자바 소프트웨어 스택의 모든 요소가 훨씬 복잡해지면서 성능 예측이 더 어려워졌고, 그에 비례해 측정의 중요성도 커졌다.

프로파일링 도구: 최적화 노력을 어디에 집중해야 할지 찾는 데 도움을 준다.

* 개별 메소드의 소비 시간과 호출 횟수같은 런타임 정보 제공
* 집중할 곳은 물론 알고리즘을 변경해야 한다는 사실을 알려주기도 함

[\[Java\] JMH(Java Microbenchmark Harness) 로 성능 벤치마킹](https://velog.io/@adduci/Java-JMHJava-Microbenchmark-Harness-%EB%A1%9C-%EC%84%B1%EB%8A%A5-%EB%B2%A4%EC%B9%98%EB%A7%88%ED%82%B9)

[Datadog's Continuous Profiling | Datadog](https://www.datadoghq.com/dg/apm/profiler/profiler-general/?utm\_source=advertisement\&utm\_medium=search\&utm\_campaign=dg-google-profiler-apac-bestprofiler\&utm\_keyword=%2Bcode%20%2Bprofiling%20%2Btools\&utm\_matchtype=b\&utm\_campaignid=15424167999\&utm\_adgroupid=130727584735\&gclid=CjwKCAiAleOeBhBdEiwAfgmXfw68pTB7KMQiOlM1SNpRJYFPi0z0ti0-OyRM2A4uVFCDcWlDKVlGyBoCf6IQAvD\_BwE)

[IntelliJ IDEA Profiling Tools](https://lp.jetbrains.com/intellij-idea-profiler/?source=google\&medium=cpc\&campaign=14001267836\&term=java%20performance%20profiling\&content=535405350407\&gclid=CjwKCAiAleOeBhBdEiwAfgmXfx-qM7DZWdukf6nugFCyvVwMvW4pK8JPyzOei3PhBIhvgzenNBG-yxoCQm4QAvD\_BwE)

</details>



