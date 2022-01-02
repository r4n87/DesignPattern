#Design Pattern
1. [Singleton](#1-싱글톤-Singleton)
2. [Factory Method](#2-팩토리-메서드-Factory-Method)
3. [Abstract Factory](#3-추상-팩토리-Abstract-Factory)
4. [Builder](#4-빌더-Builder)
5. [Prototype](#5-프로토타입-Prototype)
6. [Adapter](#6-어댑터-Adapter)
7. [Bridge](#7-브릿지-Bridge)


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


## 4. 빌더 Builder
* Creational - Object

### 목적
* 쉽게 변할 수 있는 알고리즘 기반의 object를 다양하게 생성할 수 있게 함

### Use When
* 생성 시 runtime control이 필요할 때
* 다양한 생성 알고리즘이 필요할 때
* 오브젝트 생성 알고리즘이 시스템으로부터 분리되어야 할 때
* 새로 생성 시 코어 알고리즘의 변경이 없어야 할 

### 특징
* 인터페이스가 자주 변할 때 사용하지 않는 것이 좋음
* composite object or data structure 생성에 좋음
* vs. Abstract Factory: 빌더는 step-by-step, AF는 결과가 바로 나옴

### 예시
* POS: 주문 기계, 주문을 받아서 전달하는 역할을 함
* 점원: 주문을 전달 받아 커피를 만듦
* 기계: 기계에 따라 다른 추출 방식의 커피

```
public class Pos {
    private CoffeeMachine coffeeMachine;
    
    public void setCoffeeMachine(CoffeeMachine ce) {
        coffeeMachine = ce;
    }
    
    public Coffee getCoffee() {
        return coffeeMachine.getCoffee();
    }
    
    public void makeCoffee() {
        coffeeMachine.makeNewCoffee();
        coffeeMachine.getWater();
        coffeeMachine.adjustTemp();
        coffeeMachine.getShots();
        coffeeMachine.pourIntoCup();
    }
}
```

```
public abstract class CoffeeMachine {
    protected Coffee coffee;
    protected String customer;
    protected String type;
    
    public Coffee getCoffee() {
        return coffee;
    }
    
    pubic void makeNewCoffee() {
        coffee = new Coffee(customer, type);
    }
    
    public abstract void getWater();
    public abstract void adjustTemp();
    public abstract void getShots();
    public abstract void pourIntoCup();
}
```
```
public class Coffee {
    private String type;
    private String customer;

    private int cc;
    private int temperature;
    private String bean;
    private int shotCnt;
    private String cup;

    
    Coffee (String customer, String type) {
        this.customer = customer;
        this.type = type;
    }
    
    public void setCC(int c) { this.cc = c; }
    public void setTemperature(int t) { this.temperature = t; }
    public void setShot(String b, int s) { 
        this.bean = b; 
        this.shotCnt = s;
    }
    public void setCup(String c) { this.cup = c; } 
    
    public String getCustomer() { return customer; }
    public String getType() { return type; }
}
```
```
public class HandDrip extends CoffeeMachine {
    HandDrip (String customer) {
        super.customer = customer;
        super.type = "Hand Drip Columbia";
    }
    
    public void getWater() {
        coffee.setCC(300);
    }
    
    public void adjustTemp() {
        coffee.setTemparature(70);
    }
    
    public void getShots() {
        coffee.setShot("Columbia", 2);
    }
    
    public void pourIntoCup() {
        coffee.setCup("Mug");
    }
}
```
```
public class ColdBrew extends CoffeeMachine {
    ColdBrew (String customer) {
        super.customer = customer;
        super.type = "Cold Brew Guatemala";
    }
    
    public void getWater() {
        coffee.setCC(400);
    }
    
    public void adjustTemp() {
        coffee.setTemparature(5);
    }
    
    public void getShots() {
        coffee.setShot("Guatemala", 2);
    }
    
    public void pourIntoCup() {
        coffee.setCup("Glass");
    }
}
```

```
public class CoffeeExample {
    public static void main(String[] args) {
        Pos pos = new Pos();
        CoffeeMachine cfmc = new CoffeeMachine("Jennie");
        
        pos.setCoffeeMachine(cfmc);
        pos.makeCoffee();
        Coffee completedHandDrip = pos.getCoffee();
        System.out.println(completedHandDrip.getType() + 
            " is completed and ready for delivery to " +
            completedHandDrip.getCustomer());        
    }
}
```

![Untitled Diagram drawio (2)](https://user-images.githubusercontent.com/82352179/147398352-d86bbe23-6638-4f99-8594-ed939ac29062.png)


## 5. 프로토타입 Prototype
* Creational - Object

### 목적
* 생성할 객체들의 타입이 프로토타입인 인스턴스로부터 결정되도록 함* 
* 인스턴스는 새 객체를 만들기 위해 자신을 복제하는 패턴

### Use When
* 비슷한 객체를 자주 생성해야 할 때
* 객체 생성 시마다 드는 비용 낭비를 막으려고 할 

### 특징
* 객체를 수정하거나 테스트할 때 매번 새로운 객체를 생성하지 않음
* 기존에 만들어놓은 Prototype을 복제하고 필요한 부분만 수정
* 깊은 복사와 얕은 복사에 주의하여 활용해야 함

### 예시
* 원형
* 지름 사이즈가 변하는 원형 구현

```
public interface Shape extends Cloneable {
    void isType();
}
```

```
public class Circle implements Shape {
    private int startX, StartY, diameter;
    public static final int pie = 3;
    
    public Circle(int startX, int startY, int diameter) {
        this.startX = startX;
        this.startY = startY;
        this.diameter = diameter;
    }
    
    public int getStartX() {
        return startX;
    }
    
    public void setStartX(int startX) {
        this.startX = startX;
    }
    
    public int getStartY() {
        return startY;
    }
    
    public void setStartY(int startY) {
        this.startY = startY;
    }
    
    public int getDiameter() {
        return diameter;
    }
    
    public void setDiameter(int diameter) {
        this.diameter = diameter;
    }
    
    public Circle copy() throws CloneNotSupportedException {
        Circle circle = (Circle) clone();
        return circle;
    }
    
    @Override
    public void isType() {
        System.out.println("This circle is ");
        System.out.println("X: " + startX + ", Y: " + startY + ", 지름: " + diameter);
        System.out.println(", 너비: " + 2/diameter * pie);
    }
}
```
```
public class Main {
   public static void main(String[] args) throws CloneNotSupportedException {
      Circle circle1 = new Circle(3,5,10);
      Circle circle2 = circle1.copy();
      circle2.setStartX(circle1.getStartY());
      circle2.setStartY(circle1.getStartY());
      
      circle1.isType();
      circle2.isType();
   }
}
```

![Untitled Diagram drawio (1)](https://user-images.githubusercontent.com/82352179/147398235-85216317-b406-4402-b5c6-1ff8e0cc2337.png)

## Reference
Libi의 블로그 프로토타입 패턴 https://sorjfkrh5078.tistory.com/

## 6. 어댑터 Adapter
* Structural - Class, Object
* = Wrapper

### 목적
* 공통 오브젝트를 생성하기 위해서 다른 인터페이스들을 사용할 수 있도록 함

### Use When
* 클래스가 해당 인터페이스를 쓸 수 있는 조건이 되지 않을 때, 쓸 수 있도록 함

### 사용
* 클라이언트 - (method 사용 요청) -> adapter
* adapter - (요청 전달) -> adaptee
* adapter - (결과 전달) -> client
* Object Adapter: composition and delegation
* Class Adapter: inheritance

### 구현
### Adaptee Class
```
public class Bread {
    private String flavor;
    
    public Bread(String flavor) {
        this.flavor = flavor;
    }
    
    public makeBread() {
        System.out.println(flavor + "맛이 나는 빵을 만들어요.");
    }
}
```
### Target Interface
```
public interface Bake {
    public abstract void adjustTemp();
    public abstract void adjustTime();
}
```
### Adapter Class
```
public class BakeBread extends Bread implements Bake {
    public BakeBread(String flavor) {
        super(flavor);
    }
    
    @Override
    public void adjustTemp() {
        System.out.println("빵을 굽기 위한 온도인 60도로 세팅합니다.");
        makeBread();
    }
    
    @Override
    public void adjustTime() {
        System.out.println("빵을 굽기 위한 시간인 30분으로 세팅합니다.");
        makeBread();
    }
}
```

### 다이어그램
![Untitled Diagram drawio (4)](https://user-images.githubusercontent.com/82352179/147877819-20274dea-d7ce-4d03-a809-26b4fbc53bf2.png)


## 7. 브릿지 Bridge
* Structural - Object

### 목적
* 커플링을 막기 위해 추상 오브젝트 구조를 독립적으로 정의함

### Use When
* 추상화와 구현이 컴파일 시점에 묶이지 않아야 할 때
* 추상화와 구현이 독립적으로 확장가능해야 할 때
* 클라이언트로부터 구현에 대한 디테일을 감추어야 할 때
* 기능 클래스 계층과 구현 클래스 계층을 연결할 때

### Adapter와의 비교
* 구현에 대한 디테일을 감춘다는 점에서 비슷함
* 어댑터 패턴은 관련 없는 컴포넌트들이 같이 일하도록 만듦 -> Applied to systems after they'res designed
* 브릿지 패턴은 추상화와 구현이 독립적으로 움직일 수 있도록 -> used up-front in a design
* 어댑터 패턴은 하나의 인터페이스만 추상화함
* 브릿지 패턴은 구현과정으로 부터 복잡한 엔티티를 추상화함

### 구현
### 기능 클래스 계층의 상위 클래스
```
public class Tv {
    private TvImpl impl;
    
    public Tv(TvImpl impl) {
        this.impl = impl;
    }
    
    public void turnOn() {
        impl.rawTurnOn();
    }
    
    public void setChannel() {
        impl.rawSetChannel();
    }
    
    public void turnOff() {
        impl.rawTurnOff();
    }
    
    public final void showTv() {
        turnOn();
        setChannel();
        turnOff();
    }
}
```
### 기능 클래스 계층의 하위 클래스
```
public class DualTv extends Tv {
    public DualTv(TvImpl impl) {
        super(impl);
    }
    
    public void showDualTv() {
        turnOn();
        for(int i = 0; i < 2; i++) {
            setChannel();
        }
        turnOff();
    }
}
```
### 구현 클래스 계층의 최상위 
```
public interface TvImpl {
    public void rawTurnOn();
    public void rawSetChannel();
    public void rawTurnOff();
}
```
### 구현 클래스 계층의 하위
```
public class 3DTvImpl implements TvImpl {
    private boolean 3DOn;
    private int count;
    private Channel channel;
    
    public 3DTvImpl(boolean 3DOn) {
        this.3DOn = 3DOn;
    }
    
    @Override
    public void rawTurnOn() {
        System.out.println("3D turn on");
    }
    
    @Override
    public void rawSetChannel() {
        int channel = getChannel();
        System.out.println("3D Channel number is " + channel + ".");
    }
    
    @Override
    public void rawTurnOff() {
        System.out.println("3D turn off");
    }
    
    private int getChannel() {
        for(int i = 0; i < count; i++) {
            if("3D".equals(channel.getType())) {
                return channel.getNumber();
            }
        }
    }
}
```

### 다이어그램
![Untitled Diagram drawio (3)](https://user-images.githubusercontent.com/82352179/147877494-e56f3f82-9003-4769-9b75-664b05ac4d41.png)

## Reference
https://lee1535.tistory.com/72?category=819409
