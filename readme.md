## Design Pattern
</br>

### Singleton Pattern

> 애플리케이션에서 인스턴스를 하나만 만들어 사용하기 위한 패턴

</br>

인스턴스 : 객체가 클래스 타입으로 선언되어 메모리에 할당되는 것

</br>

</br>

#### 장점

1. 인스턴스를 여러개 만들지 않아 자원(메모리) 낭비를 막을 수 있다
2. 동일한 인스턴스를 사용하기 때문에 여러번 인스턴스를 만들지 않아 코드를 간결하게 한다 -> 가독성 좋다
3. 일종의 전역 인스턴스이기 때문에 다른 클래스의 인스턴스들이 데이터를 공유하기 쉽다(단점이 되기도 한다)(static 전역 변수와의 차이는? static 변수는 프로그램이 실행되면서 끝날때까지 메모리를 할당하고 있기 때문에 좋지 않다)
4. 하나의 인스턴스만 필요한 경우의 프로그램에서 버그를 막을 수 있다
5. 두번째 이용시부터 로딩 시간이 현저하게 줄어든다

</br>

</br>


#### 단점

1. 싱글톤 인스턴스가 너무 많은 일을 하거나 많은 데이터를 공유할 경우 다른 클래스의 인스턴스들과 결합도가 높아져 "개방-폐쇄 원칙"에 위배된다(객체 지향적 프로그래밍 아님)

2. 따라서 수정 및 테스트가 어려워진다

3. 멀티 스레딩 환경에서 동기화 처리를 하지 않으면 싱글톤 인스턴스가 두 개 이상 생길 수 있다

</br>

</br>

#### 사용 예

DBCP(DataBase Connection Pool)처럼 공통된 객체를 여러개 생성해서 사용해야 하는 경우

쓰레드풀, 캐시, 대화 상자, 사용자 설정, 로그 기록 객체, 레지스터리 설정

</br>

</br>

#### 멀티 스레딩 환경에서 singleton 사용하기

getInstance() 메소드는 외부에서 호출해야 하므로 public, 나머진 전부 private

singleton 변수와 getInstance() 메소드는 static

</br>

* 가장 기본적인 singleton

```Java
public class Singleton {
  private static Singleton singleton;

  private Singleton() {}

  public static Singleton getInstance() {
    if (singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

**멀티 스레드 환경에서 두 가지 이상의 스레드가 동시에 접근하게 될 경우 두 개의 인스턴스를 만들 수 있다.** 이는 singleton 패턴에 어긋난다

</br>

</br>

* Lazy Initialization

synchronized 키워드를 이용해 서로 다른 스레드의 동시 접근을 막는다

```Java
public class Singleton {
  private static Singleton singleton;

  private Singleton() {}

  public static synchronized Singleton getInstance() {
    if (singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

멀티스레드 액세스에 잘 작동한다. 하지만 동기화는 메소드의 첫 번째 호출에만 하면 되는데, 위의 코드는 메소드 호출시 매번 동기화동기화 작업을 하여 **시간이 오래 걸려(100배) 비효율 적**이다

</br>

</br>

* Lazy Initialization 변형

```Java
public class Singleton {
  private static Singleton singleton;

  private Singleton() {}

  public static Singleton getInstance() {
    if (singleton == null) {
      synchronized(Singleton.class) {
      singleton = new Singleton();
      }
    }
    return singleton;
  }
}
```

메소드 단위의 동기화를 피하기 위해 이와 같은 코드를 작성하였다. 하지만 이 방법은 첫번째 방법과 마찬가지로 **멀티 스레드 환경에서 두 가지 인스턴스를 만들 수 있다**

</br>

</br>

* DCL(double checked locking)


double-checked locking 실패는 JVM의 버그 때문이 아니고 현재의 **자바 플랫폼 메모리 모델 때문**이다. 메모리 모델은 "난잡한 작성"을 허용하고 이것이 이디엄이 실패하는 주요 이유이다.



이 문제를 설명하기 위해서, //3행을 다시한번 살펴보아라. 이 코드는 `Singleton` 객체를 만들고 객체를 참조하기 위해서 `instance` 변수를 초기화한다. **이 코드의 문제는 `instance` 변수가`Singleton` 생성자의 바디가 실행하기 전에 non-`null` 이 될 수 있다는 점이다.**

</br>

1. Thread 1은 `getInstance()` 메소드로 들어간다.
2. Thread 1은 //1의 `synchronized` 블록으로 들어간다. `instance`가 `null`이기 때문이다.
3. Thread 1은 //3 으로 가서 non-`null` 인스턴스를 만든다. 생성자가 실행하기 전이다.
4. Thread 1은 thread 2에 선점된다.
5. Thread 2는 인스턴스가 `null`인지를 점검한다. null이 아니기 때문에, thread 2는 `instance` 레퍼런스를 완전히 만들어졌지만 부분적으로 초기화된 `Singleton` 객체로 리턴한다.
6. Thread 2는 thread 1에 선점된다.
7. Thread 1은 생성자를 실행함으로서 `Singleton` 객체의 초기화를 완료하고 레퍼런스를 리턴한다.

</br>

thread 2는 한 객체를 리턴하는데, 그 객체의 생성자는 실행되지 않았다.



**아래 내용은 참조만...**

인스턴스가 생성되었는지만 확인하고, 생성되지 않았을 때만 동기화한다(동기화 영역을 줄여준다). 이론적으로는 문제가 없지만 다중 프로세서에서 다른 CPU가 항상 lock이 걸린다는 것. 이유는 자바 플랫폼 메모리 모델 때문

JLS(Java Language Specification)을 참고할 때, 변수가 volatile로 선언되면 실행 순서가 일관적인 것으로 여겨지며, 재배치(reordering)이 일어나지 않는다. Peter Haggar은 두 가지 문제를 지적하고 있다. 첫번째는 순서 일관성의 문제가 아니라 최적화를 통해 코드가 옮겨지는 문제, 두번째는 많은 JVM이 volatile에 대한 순서 일관성조차 제대로 구현하고 있지 않다(JDK 1.5 이전 버전, 이후는 잘 작동한다고 함). C에선 volatile을 사용해 DCL 문제를 해소할 수 있지만, Java는 그렇지 않다

```Java
public class Singleton {
  private volatile static Singleton singleton;

  private Singleton() {}

  public static Singleton getInstance() {
    if (singleton == null) {
      synchronized (Singleton.class) {	//1
        if (singleton == null) {		//2
          singleton = new Singleton();	//3
        }
      }
    }
    return singleton;
  }
}
```

</br>

</br>

* volatile

  인스턴스를 필요할 때 생성하지 말고, 처음부터 만들어 버린다

```Java
public class Singleton {
  // 여기서 volatile 
  private volatile static Singleton singleton = new Singleton();

  private Singleton() {}

  public static Singleton getInstance() {
    return singleton;
  }
}
```

static 변수 이기 때문에 프로그램이 실행되고 끝날때까지 인스턴스가 메모리에 있게 된다.(인스턴스를 사용하지 않더라도) 멀티 스레드 환경에서 동작은 하지만 best는 아니다. - **메모리 비효율**

</br>

</br>

***volatile?***

멀티 스레딩 환경에서 동기화 해주는 키워드. 좀 더 구체적으로 컴파일러가 특정 변수에 대해 옵티마이져가 캐싱을 적용하지 못하게 한다

**synchronized와의 차이는 synchronized는 작업 자체를 원자해버리지만, volatile은 특정 변수에 대해서만 최신 값을 제공한다**

volatile 키워드를 사용하면 자바의 일종의 최적화인 리오더링(보통 컴파일 과정에서 일어나며, 프로그래머가 만들어낸 코드는 컴파일 될 때 좀더 빠르게 실행될 수 있도록 조작이 가해져 최적하됨)을 회피하여 읽기와 쓰기순서를 보장. 멀티스레딩을 쓰더라도 uniqueInstance변수가 Singleton 인스턴스로 초기화 되는 과정이 올바르게 진행되도록 할 수 있다

**하지만 JVM이 순차적 영속성을 정확히 고려한 volatile을 구현하지 않는다**

</br>

</br>

* Lazy Holder

```Java
public class Singleton {
  private Singleton() {}

  private static class SingletonHolder {
    private static final Singleton singleton = new Singleton();
  }

  public static Singleton getInstance() {
    return SingletonHolder.singleton;
  }
}

```

중첩 클래스를 이용해 Holder를 사용한 기법으로 가장 best. getInstance 메소드가 호출되기 전까지 Singleton 인스턴스는 생성되지 않는다(**지연된 초기화 -> 메모리 효율 좋음**). 또한 synchronized를 사용하지 않아 **성능 문제가 없다**. 그리고 최신 JVM은 클래스를 초기화하기 위한 필드 접근은 동기화된다. 초기화되고 나면 코드를 바꿔서 앞으로의 필드 접근에는 어떤 동기화나 검사도 필요하지 않게 된다. 그러므로 **초기화 된 이후에 getInstance()메소드가 호출된다고 하더라도 인스턴스는 생성되지 않는다**(JVM에서 클래스를 로딩하고 초기화할 때 원자성을 보장하기 때문)

</br>

1. final은 한번 초기화 되면 값을 변경할 수 없다.

2. **static은 Class가 로딩되는(즉, Access되는) 시점에 해당 객체가 JVM의 Class Area에 저장된다.**

3. Inner class를 사용함으로서 SingletonHolder.INSTANCE 이 부분이 실행되기 전까지는 2.의 법칙에 의해서 Load 되지 않는다.

</br>

holder안에 선언된 instance가 static이기 때문에 class 로딩 시점에 한번만 호출될 것이며, final을 사용해 다시 값이 할당되지 못하도록 만든 방법이다

</br>

</br>

volatile에 대해 좀 더 알아 볼 필요 있겠다

- volatile을 사용한 변수 (1.5이상): 변수 접근까지에 대해 모든 변수들의 상황이 업데이트 되고, 변수가 업데이트된다.
- synchronziation을 사용한 연산: synch블락 전까지의 모든 연산이 업데이트 되고, synch안의 연산이 업데이트된다.



마지막으로 volatile과 synchronization을 살펴보자. 아래의 코드가 이해를 도와줄 거라고 생각한다. i와 j를 보고 연산에 어떤 차이가 있을지 생각해봐라. 어느 변수가 멀티쓰레드 환경에서 문제가 될까?

```java
volatile int i;

i++;

int j;

synchronized { j++; }

volatile vs synchronized

```



대략 감이 잡힌다면 정말 센스 만점인 사람이다. 답은 i가 문제가 될 수 있고, j는 괜찮다는 거다. 왜냐면 i++ 이란 문장은 read i to temp; add temp 1 ; save temp to i; 라는 세개의 문장으로 나뉘어지기 때문이다. 따라서, read나 write하나만 완벽히 실행되도록 도와주는 volatile은 2번 문장이 3개로 나뉘어 질 경우에 다른 쓰레드가 접근하면 문제가 생길 수가 있다. 하지만, synchronized는 그 블럭안에 모든 연산이 방해받지 않도록 보장해주기에 j는 제대로 업데이트가 된다.

<br>

<br>

***참고***

바로 아래 링크에 안되는 이유 제일 잘 설명 되어 있음

http://gampol.tistory.com/entry/Double-checked-locking%EA%B3%BC-Singleton-%ED%8C%A8%ED%84%B4

http://blog.daum.net/smufu/3

http://asfirstalways.tistory.com/335#recentComments

http://gampol.tistory.com/m/entry/Double-checked-locking%EA%B3%BC-Singleton-%ED%8C%A8%ED%84%B4

http://kwanseob.blogspot.kr/2012/08/java-volatile.html