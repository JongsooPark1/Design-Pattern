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

  private Singleton() {

  }

  static Singleton getInstance() {
    if (singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

두 가지 이상의 스레드가 동시에 접근하게 될 경우 두 개의 인스턴스를 만들 수 있다. 이는 singleton 패턴에 어긋난다

* Lazy Initialization

synchronized 키워드를 이용해 서로 다른 스레드의 동시 접근을 막는다

```Java
public class Singleton {
  private static Singleton singleton;

  private Singleton() {

  }

  public static synchronized Singleton getInstance() {
    if (singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

* DCL(double checked locking)

인스턴스가 생성되었는지만 확인하고, 생성되지 않았을 때만 동기화한다(동기화 영역을 줄여준다). Lazy Initialization의 단점이 느리다는 것(100배)과, 항상 Lock이 걸리는 것이 었는데, 이를 해결해준다

```Java
public class Singleton {
  private static volatile Singleton singleton = new Singleton();

  private Singleton() {

  }

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

* violate
