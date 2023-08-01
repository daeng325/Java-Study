# item81. wait와 notify보다는 동시성 유틸리티를 애용하라.

item50에서는 wait와 notify를 올바르게 사용하는 방법을 안내했다. 지금은 wait와 notify를 사용해야 할 이유가 많이 줄었다. 자바5에서 도입된 고수준의 동시성 유틸리티가 이전이라면 wait와 notify로 하드코딩해야 했던 전형적인 일들을 대신 처리해주기 때문이다.&#x20;

wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.&#x20;



java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다. 바로 실행자 프레임워크, 동시성 컬렉션(Concurrent collection), 동기화 장치(synchronizer)다. 실행자 프레임워크는 item80에서 가볍게 살펴보았고, 동시성 컬렉션과 동기화 장치는 이번 아이템에서 살펴본다.&#x20;



## 동시성 컬렉션(Concurrent Collection)

동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다(item 79). 따라서 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.&#x20;



동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메소드를 원자적으로 묶어 호출하는 일 역시 불가능하다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메소드들이 추가되었다. 이 메소드들은 아주 유용해서 자바8에서는 일반 컬렉션 인터페이스에도 디폴트 메소드 (item 21) 형태로 추가되었다.&#x20;

예를 들어 Map의 putIfAbsent(key, value) 메소드는 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 집어넣는다. 그리고 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다. 이 메소드 덕에 스레드 안전한 정규화 맵(cnonicalizing map)을 쉽게 구현할 수 있다. 다음은 String.intern ([https://velog.io/@jsj3282/String.intern](https://velog.io/@jsj3282/String.intern)) 의 동작을 흉내 내어 구현한 메소드이다.&#x20;

```java
// Map의 default method인 putIfAbsent
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value); // put javadoc에 key에 대한 mapping이 없으면 null을 반환한다고 적혀있음.
    }

    return v;
}

// ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아님.
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}

```

아직 개선할 게 남았다. ConcurrentHashMap은 get 같은 검색 기능에 최적화되었다. 따라서 get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하면 더 빠르다.&#x20;

{% code overflow="wrap" %}
```java
// ConcurrentMap으로 구현한 동시성 정규화 맵 - 더 빠름!
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s); // 얘 자체만 쓰는 게 느리면 얘를 왜 만들었는지? 아 synchronized가 되어 있는 거지. 여러 기본 동작을 하나의 원자적 동작으로 묶는댔음.
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```
{% endcode %}



ConcurrentHashMap은 동시성이 뛰어나며 속도도 무척 빠르다. 내 컴퓨터에서 이 메소드는 String.intern보다 6배나 더 빠르다. (하지만 String.intern에는 오래 실행되는 프로그램에서 메모리 누수를 방지하는 기술도 들어가 있음을 감안하자). 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들어버렸다. 대표적인 예로, 이제는 Collections.synchronizedMap 보다는 ConcurrentHashMap을 사용하는 게 훨씬 좋다. 동기화된 맵을 동시성 맵으로 교체하는 것만으로 동시성 애플리케이션의 성능은 극적으로 개선된다.&#x20;



컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 (즉, 차단되도록) 확장되었다.&#x20;

예를 들어, Queue를 확장한 BlockingQueue에 추가된 메소드 중 take는 큐의 첫 원소를 꺼낸다. 이 때 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 이런 특성 덕에 BlockingQueue는 작업 큐(Producer-Consumer Queue)로 쓰기에 적합하다. 작업 큐는 하나 이상의 producer 스레드가 작업 (work)을 큐에 추가하고, 하나 이상의 consumer 스레드가 큐에 있는 작업을 꺼내 처리하는 형태다. 짐작하다시피 ThreadPoolExecutor를 포함한 대부분의 Executor Service(item 80) 구현체에서 이 BlockingQueue를 사용한다.&#x20;



## 동기화 장치(Synchronizer)

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다. 가장 자주 쓰이는 동기화 장치는 ContDownLatch와 Semaphore다. CyclicBarrier와 Exchanger는 그보다 덜 쓰인다. 그리고 가장 강력한 동기화 장치는 바로 Phaser다.&#x20;



CountDownLatch(latch; 걸쇠)는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. CountDownLatcn의 유일한 생성자는 int 값을 받으며, 이 값이 latch의 countDown 메소드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.&#x20;



### CountDownLatch Example

동시 실행 시간을 재는 간단한 프레임워크&#x20;

* 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 프레임워크 구축
* 메소드 하나로 구성, 이 메소드는 동작들을 실행할 executor와 동작을 몇 개나 수행할 수 있는지를 나타내는 동시성 수준(concurrency)을 매개변수로 받는다.&#x20;
* 타이머 스레드가 시계를 시작하기 전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다.
* 마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 '시작 방아쇠'를 당겨 작업자 스레드들이 일을 시작한다.
* 마지막 작업자 스레드가 동작을 마치자마자 타이머 스레드는 시계를 멈춘다.&#x20;

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {
            // 타이머에게 준비를 마쳤음을 알린다.
            ready.countDown();
            try {
                // 모든 작업자 스레드가 준비될 때까지 기다린다.
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                // 타이머에게 작업을 마쳤음을 알린다.
                done.countDown();
            }
        });
    }

    ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await(); // 모든 작업자가 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
}
```

#### 설명&#x20;

ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다. 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다. 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown을 호출하여 작업자들을 깨운다. 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다. done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호춣하면 열린다. 타이머 스레드는 done 래치가 열리자마자 깨어나 종료 시각을 기록한다.&#x20;



#### 몇 가지 추가 세부사항&#x20;

* time 메소드에 넘겨진 executor는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메소드는 결코 끝나지 않을 것이다. 이런 상태를 Thread Starvation Deadlock이라 한다.&#x20;
* InterruptedException을 캐치한 작업자 스레드는 Thread.currentThread().interrupt() 관용구를 사용해 인터럽트를 되살리고 자신은 run 메소드에서 빠져나온다. 이렇게 해야 실행자가 인터럽트를 적절하게 처리할 수 있다. &#x20;
* 시간 간격을 잴 때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용하자. System.nanoTinme은 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.&#x20;
* 이 예제의 코드는 작업에 충분한 시간(예: 1초 이상)이 걸리지 않는다면 정확한 시간을 측정할 수 없을 것이다. 정밀한 시간 측정은 매우 어려운 작업이라, 꼭 해야 한다면 jmh 같은 특수 프레임워크를 사용해야 한다.&#x20;



앞서  보여준  CountDownLatch 3개는 CyclicBarrier(혹은 Phaser) 인스턴스 하나로 대체할 수 있다. 코드가 더 명료해지기는 하겠지만 아마도 이해하기는 더 어려울 것.



그럼에도 불구하고 wait & notify를 써야 하는 경우가 있다면 아래 확인

<details>

<summary>wait &#x26; notify Example</summary>

새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 써야 한다. 하지만 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있을 것이다. wait 메소드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다. 락 객체의 wait 메소드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다. wait를 사용하는 표준 방식은 다음과 같다.&#x20;



```java
synchronized (obj) {
    while(<조건이 충족되지 않았다>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다. 
    }
    ... // 조건이 충족됐을 때의 동작을 수행한다. 
}
```

wait 메소드를 사용할 때는 반드시 대기 반복문 (wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자. 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.&#x20;

대기 전에 조건을 검사하여 조건이 이미 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다. 만약 조건이 이미 충족되었는데 스레드가 notify(혹은 notifyAll) 메소드를 먼저 호출한 후 대기 상태로 빠지게 되면 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.&#x20;

한편,  대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패 (safety failure)를 막는 조치다. 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위함이 있다. 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 몇 가지 있으니, 다음이 그 예다.

* 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.&#x20;
* 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메소드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
* 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
* 대기 중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 있다. 허위 각성(spurious wakeup)이라는 현상이다.&#x20;



이와 관련하여 notify와 notifyAll 중 무엇을 선택하느냐 하는 문제도 있다. (notify는 스레드 하나만 깨우며, notifyAll은 모든 스레드를 깨운다.) 일반적으로 언제나 notifyAll을 사용하라는 게 합리적이고 안전한 조언이 될 것이다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다. 다른 스레드까지 깨어날 수도 있긴 하지만, 그것이 여러분 프로그램의 정확성에는 영향을 주지 않을 것이다. 깨어난 스레드들은 기다리던 조건이 충족되었는지 확인하여, 충족되지 않았다면 다시 대기할 것이다.

모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화할 수 있다.

하지만 이상의 전제조건들이 만족될지라도 notify 대신 notifyAll을 사용해야 하는 이유가 있다. 외부로 공개된 객체에 대해 실수로 혹은 악의적으로 notify를 호출하는 상황에 대비하기 위해 wait를 반복문 안에서 호출했듯, notify 대신 notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait를 호출하는 공격으로부터 보호할 수 있다. 그런 스레드가 중요한 notify를 삼켜버린다면 꼭 깨어났어야 할 스레드들이 영원히 대기하게 될 수 있다.&#x20;

</details>



{% hint style="info" %}
코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀) 없다. java.util.concurrent의 고수준 동시성 유틸리티를 사용하자.&#x20;
{% endhint %}

