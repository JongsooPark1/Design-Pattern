## Design Pattern
---

### Singleton Pattern

> 애플리케이션에서 인스턴스를 하나만 만들어 사용하기 위한 패턴

#### 장점

1. 인스턴스를 여러개 만들지 않아 자원(메모리) 낭비를 막을 수 있다

2. 동일한 인스턴스를 사용하기 때문에 여러번 인스턴스를 만들지 않아 코드를 간결하게 한다 -> 가독성 좋다

3. 일종의 전역 인스턴스이기 때문에 다른 클래스의 인스턴스들이 데이터를 공유하기 쉽다(단점이 되기도 한다)(static 전역 변수와의 차이는? static 변수는 프로그램이 실행되면서 끝날때까지 메모리를 할당하고 있기 때문에 좋지 않다)

4. 하나의 인스턴스만 필요한 경우의 프로그램에서 버그를 막을 수 있다

5. 두번째 이용시부터 로딩 시간이 현저하게 줄어든다


#### 단점

1. 싱글톤 인스턴스가 너무 많은 일을 하거나 많은 데이터를 공유할 경우 다른 클래스의 인스턴스들과 결합도가 높아져 "개방-폐쇄 원칙"에 위배된다(객체 지향적 프로그래밍 아님)

2. 따라서 수정 및 테스트가 어려워진다

3. 멀티 스레딩 환경에서 동기화 처리를 하지 않으면 싱글톤 인스턴스가 두 개 이상 생길 수 있다

#### 사용 예

DBCP(DataBase Connection Pool)처럼 공통된 객체를 여러개 생성해서 사용해야 하는 경우

쓰레드풀, 캐시, 대화 상자, 사용자 설정, 로그 기록 객체, 레지스터리 설정

#### 멀티 스레딩 환경에서 singleton 사용하기

기존 코드

```Java
public class Singleton {
  private static Singleton singleton;

  private Singleton() {}

  static Singleton getInstance() {
    if (singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

멀티 스레드 환경에서 두 가지 이상의 스레드가 동시에 접근하게 될 경우 두 개의 인스턴스를 만들 수 있다. 이는 singleton 패턴에 어긋난다

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

동기화 작업을 메소드 단위로 하기 때문에 시간이 오래 걸려(100배) 비효율 적이다

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

메소드 단위의 동기화를 피하기 위해 이와 같은 코드를 작성하였다. 하지만 이 방법은 첫번째 방법과 마찬가지로 멀티 스레드 환경에서 두 가지 인스턴스를 만들 수 있다

* DCL(double checked locking)

인스턴스가 생성되었는지만 확인하고, 생성되지 않았을 때만 동기화한다(동기화 영역을 줄여준다). 이론적으로는 문제가 없지만 다중 프로세서에서 다른 CPU가 항상 lock이 걸린다는 것. 이유는 자바 플랫폼 메모리 모델 때문

JLS(Java Language Specification)을 참고할 때, 변수가 volatile로 선언되면 실행 순서가 일관적인 것으로 여겨지며, 재배치(reordering)이 일어나지 않는다. Peter Haggar은 두 가지 문제를 지적하고 있다. 첫번째는 순서 일관성의 문제가 아니라 최적화를 통해 코드가 옮겨지는 문제, 두번째는 많은 JVM이 volatile에 대한 순서 일관성조차 제대로 구현하고 있지 않다(JDK 1.5 이전 버전, 이후는 잘 작동한다고 함). C에선 volatile을 사용해 DCL 문제를 해소할 수 있지만, Java는 그렇지 않다

```Java
public class Singleton {
  private static volatile Singleton singleton;

  private Singleton() {}

  public static Singleton getInstance() {
    if (singleton == null) {
      synchronized (Singleton.class) {
        if (singleton == null) {
          singleton = new Singleton();
        }
      }
    }
    return singleton;
  }
}
```

* 인스턴스를 필요할 때 생성하지 말고, 처음부터 만들어 버린다

```Java
public class Singleton {
  // 여기서 volatile 
  private static volatile Singleton singleton = new Singleton();

  private Singleton() {}

  public static Singleton getInstance() {
    return singleton;
  }
}
```

static 변수 이기 때문에 프로그램이 실행되고 끝날때까지 인스턴스가 메모리에 있게 된다.(인스턴스를 사용하지 않더라도) 멀티 스레드 환경에서 동작은 하지만 best는 아니다

***volatile?***

멀티 스레딩 환경에서 동기화 해주는 키워드. 좀 더 구체적으로 컴파일러가 특정 변수에 대해 옵티마이져가 캐싱을 적용하지 못하게 한다

synchronized와의 차이는 synchronized는 작업 자체를 원자해버리지만, volatile은 특정 변수에 대해서만 최신 값을 제공한다

volatile 키워드를 사용하면 자바의 일종의 최적화인 리오더링(보통 컴파일 과정에서 일어나며, 프로그래머가 만들어낸 코드는 컴파일 될 때 좀더 빠르게 실행될 수 있도록 조작이 가해져 최적하됨)을 회피하여 읽기와 쓰기순서를 보장 멀티스레딩을 쓰더라도 uniqueInstance변수가 Singleton 인스턴스로 초기화 되는 과정이 올바르게 진행되도록 할 수 있다

하지만 JVM이 순차적 영속성을 정확히 고려한 volatile을 구현하지 않는다

* Lazy Holder

```Java
public class Singleton {
  private Singleton() {}

  private static class SingletonHolder {
    public static final Singleton singleton = new Singleton();
  }

  public static Singleton getInstance() {
    return SingletonHolder.singleton;
  }
}

```

중첩 클래스를 이용해 Holder를 사용한 기법으로 가장 best. getInstance 메소드가 호출되기 전까지 Singleton 인스턴스는 생성되지 않는다(지연된 초기화 -> 메모리 효율 좋음). 또한 synchronized를 사용하지 않아 성능 문제가 없다. 그리고 최신 JVM은 클래스를 초기화하기 위한 필드 접근은 동기화된다. 초기화되고 나면 코드를 바꿔서 앞으로의 필드 접근에는 어떤 동기화나 검사도 필요하지 않게 된다. 그러므로 초기화 된 이후에 getInstance()메소드가 호출된다고 하더라도 인스턴스는 생성되지 않는다(JVM에서 클래스를 로딩하고 초기화할 때 원자성을 보장하기 때문)


***참고***

http://asfirstalways.tistory.com/335#recentComments

http://gampol.tistory.com/m/entry/Double-checked-locking%EA%B3%BC-Singleton-%ED%8C%A8%ED%84%B4

http://kwanseob.blogspot.kr/2012/08/java-volatile.html

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