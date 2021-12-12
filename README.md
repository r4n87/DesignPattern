# 싱글톤 Singleton

* 어떠한 클래스(객체)가 유일하게 1개만 존재할 때 사용
* 서로 자원을 공유할 때 사용
* 멤버 변수를 private static, 생성자를 private, 호출을 static getInstance() 메서드로 생성

### (+) 장점
* 객체를 1번만 생성하기 때문에 메모리 낭비를 방지
* 전역 인스턴스로 다른 클래스의 인스턴스가 데이터를 공유하기 쉬움
* 인스턴스가 절대적으로 한 개만 존재하는 것을 보증할 수 있음

### (-) 단점
* 싱글톤 인스턴스가 많은 곳에서 공유될 경우, 인스턴스간 결합도가 높아져 "개방-폐쇄원칙(Open-Closed Principle)" 위배
* 멀티 쓰레드 환경에서 동기화 처리하지 않으면 인스턴스가 두 개 이상 생성될 수 있음

### Thread-safe Singleton Class
**1. Thread-safe Lazy Initialization(게으른 초기화)**
```
public class ThreadSafeLazy {
    private static ThreadSafeLazy instance;
    private ThreadSafeLazy(){}
    public static syncrhonized ThreadSafeLazy getInstance() {
        if(null == instance) {
            instance = new ThreadSafeLazy();
        }
        return instance;
    }
}
```
- 기본 Singleton의 getInstance() 메서드에 synchronized 처리를 하여 구현
- synchronized에 의하여 성능 저하가 발생하므로 권장하지 않는 방법

**2. Thread-safe Lazy Initialization + Double-checked Locking(게으른 초기화+DCL)**
```
public class LazyDouble {
    private volatile static LazyDouble instance;
    private LazyDouble(){}
    public static LazyDouble getInstance() {
        if(null == instance) {
            synchronized (LazyDouble.class) {
                if(null == instance) {
                    instance = new LazyDouble();
                }
            }
        }
        return instance;
    }
}
```
- getInstance() 메서드에 synchronized를 구현하는 것이 아니라, 인스턴스 존재 여부 체크 후에 동기화
- thread-safe + synchronized 블럭을 여러번 타지 않게 함

**3. Initialization on demand holder idiom(holder 초기화)**
```
public class Holder {
    private Holder() {}
    private static class Holder {
        public static final Holder INSTANCE = new Holder();
    }
    public static Holder getInstance() {
        return Holder.INSTANCE; 
   }
}
```
- 클래스 안에 클래스를 두어 JVM의 Class Loader Mechanism + Class Load 시점을 이용한 방법
- Holder안에 선언된 instance가 static이기 때문에 클래스 로딩 시점에 한번만 호출되며, final 객체이므로 값이 다시 할당되지 않음
- 가장 많이 사용하고 일반적인 Singleton 클래스 사용법

# Reference
Fastcampus 한 번에 끝내는 Java/Spring 웹 개발 마스터 초격차 패키지 Online.
https://jeong-pro.tistory.com/86
