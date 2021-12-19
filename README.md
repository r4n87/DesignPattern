#Design Pattern
1. [Singleton](1-싱글톤-Singleton)
2. [Factory Method](2-팩토리-메서드-Factory-Method)
3. [Abstract Factory](3-추상-팩토리-Abstract-Factory)


## 1. 싱글톤 Singleton

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

## Reference
Fastcampus 한 번에 끝내는 Java/Spring 웹 개발 마스터 초격차 패키지 Online.
https://jeong-pro.tistory.com/86


## 2. 팩토리 메서드 Factory Method
* Creational - Class

### 목적
* 오브젝트를 생성하고, 하위 클래스가 실제 생성 프로세스를 제어할 수 있도록함

### Use When
* 어떠한 클래스를 사용해서 클래스를 생성해야할 지 모를 때
* 부모 클래스가 하위 클래스에게 클래스 생성을 넘기고자 할 때

### 특징
* 하위 클래스가 어떠한 object를 생성할지 정하게 함으로써 object 생성을 캡슐화함
* 인터페이스에 object 생성에 대해 기술하나, 하위 클래스가 어떠한 클래스를 instantiation할지 정하게 함
* DIP(Dependency Inversion Principle): 상위 레벨의component는 낮은 레벨의 component에 의존하지 않음
* inheritance를 활용함

```
abstract class DonutFactory {
    protected abstract Donut makeDonut();
    public void packingDonut() {
        private final Donut donut = makeDonut();
        .
        .
        .
    }
}
```
```
class ChocoDonutFactory extends DonutFactory {
   @Override
   protected Donut makeDonut() {
       return new ChocoDonut();
   }
}
```
![hmm drawio (2)](https://user-images.githubusercontent.com/82352179/146678299-4de8b508-3de5-4a82-b34e-4176b8be1774.png)



## 3. 추상 팩토리 Abstract Factory
* Creational - Object

### 목적
* 생성 콜을 delegation하는 인터페이스를 제공하여 하위 클래스가 구체적인 object를 생성할 수 있도록 함

### Use When
* Object 생성이 시스템으로부터 독립적이어야 할 때
* Object의 그룹이 함께 쓰여야 할 때
* 상세 구현법에 대해 노출하고 싶지 않을 때

### 특징
* Factory Method보다 abstraction 레벨이 높음
* Composition & delegation을 활용함

### 장점
* 구체적인 클래스를 분리함
* product 간의 consistency를 지킬 수 있음

### 단점
* 새로운 종류의 product 생성이 어려움 (not impossible, but costly)

```
interface DonutStore {
   Donut makeDonut();
   Coffee makeCoffee();
}
```
```
public class ChocoDonutStore implements DonutStore {
    Donut makeDonut() {
        return new ChocoDonut();
    }
    
    Coffee makeCoffee() {
        return new ChocoCoffee();
    }
}
```
```
public class MilkDonutStore implements DonutStore {
    Donut makeDonut() {
        return new MilkDonut();
    }
    
    Coffee makeCoffee() {
        return new MilkCoffee();
    }
}
```
![hmm drawio (1)](https://user-images.githubusercontent.com/82352179/146678162-f1b87bc8-e3e6-4276-bf03-fb9d5a2d206c.png)


## Factory Method vs. Abstract Factory
### 공통점
* Creational Pattern
* Object의 생성을 하위 클래스가 수행할 수 있도록 함
### 차이점
* FM: 인터페이스를 정의하고, 하위 클래스가 어떠한 클래스를 instantiation할지 정하게 함
* AF: 인터페이스를 제공하나, 특정한 클래스에 대해 구체화하지 않고 "families of related or dependent objects"의 형태로 제공


