# item11. equals를 재정의하려거든 hashCode도 재정의하라.

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
