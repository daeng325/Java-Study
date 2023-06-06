# item48. 스트림 병렬화는 주의해서 적용하라.

<details>

<summary>동시성 프로그래밍 측면에서의 자바</summary>

1. 1996년부터 스레드, 동기화, wait/notify를 지원
2. 자바5부터 동시성 컬렉션인 java.util.concurrent 라이브러리와 Executor 프레임워크 지원
3. 자바7부터 고성능 병렬 분해 (parallel decom-position) 프레임워크인 fork-join 패키지 추가
4. 자바8부터 parallel 메소드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림 지원

</details>

자바로 동시성 프로그램을 작성하기가 점점 쉬워지고는 있지만, 이를 올바르고 빠르게 작성하는 일은 여전히 어려운 작업이다. 동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.



## 병렬 처리 (Parallel Operation)

병렬 처리란 멀티 코어 CPU 환경에서 하나의 작업을 분할해서 각각의 코어가 병렬적으로 처리하는 것을 말하는데, 병렬 처리의 목적은 작업 처리 시간을 줄이기 위한 것이다. 자바8부터 병렬 스트림을 제공하기 때문에 컬렉션의 전체 요소 처리 시간을 줄여 준다.



### 동시성(Concurrency)과 병렬성(Parallelism)

멀티 스레드는 동시성(Concurrency) 또는 병렬성(Parallelism)으로 실행되기 때문에 이 용어들에 대해 정확히 이해하는 것이 좋다. 이 둘은 멀티 스레드의 동작 방식이라는 점에서는 동일하지만 서로 다른 목적을 가지고 있다.&#x20;

동시성은 멀티 작업을 위해 멀티 스레드가 번갈아가며 실행하는 성질을 말하고, 병렬성은 멀티 작업을 위해 멀티 코어를 이용해 동시에 실행하는 성질을 말한다.&#x20;



### 데이터 병렬성(Data parallelism)과 작업 병렬성(Task parallelism)

데이터 병렬성

* 전체 데이터를 쪼개어 서브 데이터들로 만들고 이 서브 데이터들을 병렬 처리해서 작업을 빨리 끝내는 것
* 자바 8에서 지원하는 병렬 스트림은 데이터 병렬성을 구현&#x20;

작업 병렬성

* 서로 다른 작업을 병렬 처리하는 것
* 대표적인 예: 웹 서버(Web Server)
  * 웹 서버는 각각의 브라우저에서 요청한 내용을 개별 스레드에서 병렬로 처리한다.&#x20;



### 포크조인(ForkJoin) 프레임워크

병렬 스트림은 요소들을 병렬 처리하기 위해 백그라운드에서 포크조인(ForkJoin) 프레임워크를 사용한다.&#x20;

런타임 시에 포크조인 프레임워크가 동작

* Fork: 전체 데이터를 서브 데이터로 분리하고, 서브 데이터를 멀티 코어에서 병렬로 처리
  * 전체 데이터를 순차적으로 N(코어수) 등분하지 않고, 내부적으로 서브 요소를 나누는 알고리즘 존재
* Join: 서브 결과를 결합해서 최종 결과를 만들어냄

포크조인 프레임워크는 포크와 조인 기능 이외에 스레드풀인 ForkJoinPool을 제공한다. 각각의 코어에서 서브 요소를 처리하는 것은 개별 스레드가 해야 하므로 스레드 관리가 필요하다.&#x20;



{% code overflow="wrap" %}
```java
List<String> list = Arrays.asList(
        "홍길동", "신용권", "김자바",
        "람다식", "박병렬");

// 순차 처리
Stream<String> stream = list.stream();
// stream.spliterator().trySplit().estimateSize(); //2 
// 이 spliterator가 무한한 수의 요소를 포함하지 않는 한, trySplit()을 반복적으로 호출하면 결국 null을 반환해야 한다.
stream.forEach(EffectiveJava::print);
System.out.println();

// 병렬 처리
Stream<String> parallelStream = list.parallelStream();
parallelStream.forEach(EffectiveJava::print); // main 스레드를 포함해 ForkJoinPool(스레드풀)의 작업 스레드들이 병렬적으로 요소를 처리
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>



### 병렬 처리 성능

스트림 병렬 처리가 스트림 순차 처리보다 항상 실행 성능이 좋을까? 병렬 처리에 영향을 미치는 다음 3가지 요인을 잘 살펴보아야 한다.

#### 1. 요소의 수와 요소 당 처리 시간

컬렉션의 요소의 수가 적고 요소 당 처리 시간이 짧으면 순차 처리가 오히려 병렬 처리보다 빠를 수 있다. 병렬 처리는 스레드풀 생성, 스레드 생성이라는 추가 비용 발생하기 때문.

#### 2. 스트림 소스의 종류

ArrayList, 배열은 인덱스로 요소를 관리하기 때문에 포크 단계에서 요소를 쉽게 분리 가능해 병렬 처리 시간이 절약된다.&#x20;

반면 HashSet, TreeSet은 요소 분리가 쉽지 않고, LinkedList 역시 링크를 따라가야 하므로 요소 분리가 쉽지 않다. 따라서 이 소스들은 ArrayList, 배열보다는 상대적으로 병렬 처리가 늦다.



<details>

<summary>Effective Java 3에서 말하는 병렬처리하기 좋은 스트림 소스의 종류</summary>

대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다. ~~(int 범위, long 범위가 뭔 소린데?)~~

전체 데이터를 쪼개는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메소드로 얻어올 수 있다.&#x20;

이 자료구조들의 공통점은 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어나다는 것이다. (이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻). 하지만 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 그러면 참조 지역성이 나빠진다.&#x20;

참조 지역성이 낮으면 스레드는 데이터가 메인메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하니 보내게 된다. 따라서 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다. 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.&#x20;

</details>

#### 3. 코어(Core)의 수

싱글 코어 CPU일 경우 순차 처리가 빠르다. 병렬 스트림을 사용할 경우 스레드의 수만 증가하고 동시성 작업(Concurrent)으로 처리되기 때문이다.&#x20;



#### 4.  스트림 파이프라인의 종단 연산의 동작 방식 (Effective Java 3)

<details>

<summary>Terminal Operation</summary>

**스트림의 최종 연산(terminal operation)**

스트림 API에서 중개 연산**(intermediate operation)**을 통해 변환된 스트림은 마지막으로 최종 연산을 통해 각 요소를 소모하여 결과를 표시합니다.

즉, 지연(lazy)되었던 모든 중개 연산들이 최종 연산 시에 모두 수행되는 것입니다.

이렇게 최종 연산 시에 모든 요소를 소모한 해당 스트림은 더는 사용할 수 없게 됩니다.

&#x20;

스트림 API에서 사용할 수 있는 대표적인 최종 연산과 그에 따른 메소드는 다음과 같습니다.

&#x20;

1\. 요소의 출력 : forEach()

2\. 요소의 소모 : reduce()

3\. 요소의 검색 : findFirst(), findAny()

4\. 요소의 검사 : anyMatch(), allMatch(), noneMatch()

5\. 요소의 통계 : count(), min(), max()

6\. 요소의 연산 : sum(), average()

7\. 요소의 수집 : collect()

</details>

종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)이다. reduction은 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다.&#x20;



병렬화에 적합한 메소드

* reduce 메소드
* min, max, count, sum 메소드&#x20;
* anyMatch, allMatch, noneMatch&#x20;
  * 메소드 조건에 맞으면 바로 반환

병렬화에 적합하지 않은 메소드

* collect 메소드 (가변 축소 (mutable reduction)이라고 한다.)
  * 컬렉션들을 합치는 부담이 크기 때문&#x20;

###

### 스트림을 잘못 병렬화하면?&#x20;

#### 응답 불가 (liveness failure)

```java
public static void main(String[] args) {
    // 처음 20개의 메르센 소수
    // 메르센 수: 2^n-1 형태의 수, 여기서 n이 소수이면 메르센 수도 소수일 수도 있는데, 이 때의 수를 메르센 소수라 한다.
    primes().map(p -> BigInteger.valueOf(2).pow(p.intValueExact()).subtract(BigInteger.ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}

public static Stream<BigInteger> primes() {
    // 다음 코드는 무한 스트림을 반환하는 메소드이다. 스트림의 원소는 소수이다.
    return Stream.iterate(BigInteger.valueOf(2), BigInteger::nextProbablePrime);
}
```

위 프로그램 속도를 높이고 싶어 스트림 파이프라인의 parallel()을 호출하겠다는 생각을 했다고 치자. 이렇게 하면 성능은 어떻게 변할까? 저자의 말로는 아무것도 출력하지 못하면서 CPU는 90%나 잡아먹는 상태가 무한히 계속된다고 한다. (응답 불가: liveness failure)

이는 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다. 환경이 아무리 좋더라도 데이터 소스가 `Stream.iterate`거나 중간 연산으로 `limit`를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수가 없다.&#x20;

파이프라인 병렬화는 `limit`를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다. 그런데 이 코드의 경우 새롭게 메르센 소수를 찾을 때마다 그 전 소수를 찾을 때보다 두 배 정도 더 오래 걸린다. (원소 하나를 계산하는 비용이 대략 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다.) 그래서 이 순전무결해 보이는 파이프라인은 자동 병렬화 알고리즘이 제 기능을 못하게 마비시킨다.&#x20;



#### 안전 실패(safety failure)

스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다. 이런 오동작을 안전 실패 (safety failure)라 한다.

안전 실패는 병렬화환 파이프라인이 사용하는 mappers, filters, 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있다. Stream 명세는 이 때 사용되는 함수 객체에 관한 엄중한 규약을 정의해 놨다.&#x20;

* ex) Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고(associative), 간섭받지 않고(non-interfering), 상태를 갖지 않아야 한다(stateless).&#x20;

앞서 얘기한 메르센 소수 프로그램은 완료되더라도 출력된 소수의 순서가 올바르지 않을 수 있다. 출력 순서를 순차 버전처럼 정렬하고 싶다면 종단 연산 forEach를 forEachOrdered로 바꿔주면 된다.&#x20;



{% hint style="danger" %}
스트림 파이프라인을 마구잡이로 병렬화하면 안 된다. \
성능이 오히려 끔찍하게 나빠질 수도 있다.&#x20;
{% endhint %}





## 스트림 파이프라인 병렬화가 효과적이려면?

실제로 성능이 향상될지를 추정해보는 간단한 방법

* 스트림 안의 원소 수와 원소 당 수행되는 코드 줄 수를 곱해보자. 이 값이 최소 수십만은 되어야 성능 향상을 맛볼 수 있다.&#x20;



스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 한다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다(item 67). 조건이 잘 갖춰지면 paralle 메소드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.&#x20;



### 스트림 파이프라인 병렬화가 효과적인 사례

1\)&#x20;

π(n): n보다 작거나 같은 소수의 개수를 계산하는 함수

```java
public static long pi(long n) {
    return LongStream.rangeClosed(2, n)
    //        .parallel()
            .mapToObj(BigInteger::valueOf)
            .filter(i -> i.isProbablePrime(50))
            .count();
}
```

저자의 컴퓨터에서(쿼드코어)는 π(10^8)을 계산하는데 31초가 걸렸고, parallel()을 추가하면 9.2초로 단축되었다고 한다.&#x20;

(n이 더 크다면 레머의 공식(Lehmer's Formula)이라는 더 효율적인 알고리즘이 있다고는 함)



2\)&#x20;

무작위 수들로 이뤄진 스트림을 병렬 처리하고 싶을 때?

ThreadLocalRandom(혹은 구식인 Random)보다는 SplittableRandom 인스턴스를  이용하자. SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라 병렬화하면 성능이 선형으로 증가한다.&#x20;

ThreadLocalRandom은 단일 스레드에서 쓰고자 만들어져서, 병렬 스트림용 데이터 소스로 사용하려고 해도 SplittableRandom만큼 빠르지는 않을 것이다.&#x20;

마지막으로 그냥 Random은 모든 연산을 동기화 하기 때문에 병렬 처리하면 최악의 성능을 보일 것이다.&#x20;



{% hint style="success" %}
계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라.&#x20;

스트림을 잘못 병렬화하면 프로그램을 오동작하게하거나 성능을 급격히 떨어트린다.&#x20;

병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능 지표를 유심히 관찰하라. 그래서 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때, 오직 그럴 때만 병렬화 버전 코드를 운영 코드에 반영하라.&#x20;
{% endhint %}

