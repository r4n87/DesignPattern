# Design Pattern
1. [Singleton](#1-싱글톤-Singleton)
2. [Factory Method](#2-팩토리-메서드-Factory-Method)
3. [Abstract Factory](#3-추상-팩토리-Abstract-Factory)
4. [Builder](#4-빌더-Builder)
5. [Prototype](#5-프로토타입-Prototype)
6. [Adapter](#6-어댑터-Adapter)
7. [Bridge](#7-브릿지-Bridge)
8. [Composite](#8-컴포짓-Composite)
9. [Decorator](#9-데코레이터-Decorator)
10. [Facade](#10-파사드-Facade)
11. [Flyweight](#11-플라이웨이트-Flyweight)
12. [Proxy](#12-프록시-Proxy)
13. [Chain of Responsibility](#13-책임-연쇄-Chain-of-Responsibility)
14. [Command](#14-커맨드-Command)


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



## 8. 컴포짓 Composite
* Structural - Object

### 목적
* 오브젝트의 수직적 구조를 만들기 위해 사용
* 각 오브젝트는 독립적으로, 혹은 같은 interface를 사용하는 오브젝트 그룹으로도 사용됨

### Use When
* 오브젝트간의 수직적(상하) 구조를 나타내야 할 때
* 오브젝트들과, 합성된 오브젝트들이 하나로 취급되어야 할 때

### Decorator와의 비교
* 비슷한 구조의 diagram을 가지고 있음 -> recursive composition
* 컴포짓은 embelishment(장식) 보다 representation(표현)에 중점을 둠
* 데코레이터는 subclassing을 하지 않고 각 오브젝트에 역할을 부여함(responsibility)

### 구현
### Component 클래스
```
public abstract class KeyboardComponent {
    public void add();
    public void remove();
    public double getPrice();
    public String getName();
}
```

### Leaf 클래스
```
public class KeyCap extends KeyboardComponent {
    private double price;
    private String name;
    
    public KeyCap(double price, String name) {
        this.price = price;
        this.name = name;
    }
    
    public double getPrice() {
        return price;
    }
    
    public String getName() {
        return name;
    }
}
```
```
public class Switch extends KeyboardComponent {
    private double price;
    private String name;
    
    public Switch(double price, String name) {
        this.price = price;
        this.name = name;
    }
    
    public double getPrice() {
        return price;
    }
    
    public String getName() {
        return name;
    }
}
```

### Composite 클래스
```
public class Keyboard extends KeyboardComponent {
    private List<KeyboardComponent> components = new ArrayList<KeyboardComponent>();
    public add(KeyboardComponent component) {
        components.add(component);
    }
    
    public remove(KeyboardComponent component) [
        components.remove(component);
    }
    
    public double getPrice() {
        double result = 0;
        for(KeyboardComponent component : components) {
            result += component.getPrice();
        }
        
        return result;
    }
    
    public String getName() {
        StringBuilder sb = new StringBuilder();
        sb.append("키보드 구성품: ");
        for(KeyboardComponent component : components) {
            sb.append(component.getName());
            sb.append(" ");
        }
        
        return sb.toString();
    }
}
```

### 다이어그램
![Untitled Diagram drawio (6)](https://user-images.githubusercontent.com/82352179/148687779-e014c7bd-ea66-41e7-8f0c-757fa5321931.png)


## 9. 데코레이터 Decorator
* Structural - Object

### 목적
* 기존의 역할을 수정하기 위하여 다양한 wrapping 적용

### Use When
* 오브젝트의 역할이 수시로 변할 때
* 구체적인 구현이 역할로부터 decoupling 되어야 할 때
* 특정 기능이 오브젝트 수직적 구조가 아닐 때

### Principle
* OCP(Open-closed Principle)
* composition, delegation

### 단점
* 너무 많은 작은 클래스를 만들게 됨
* 패턴과 친숙하지 않으면 이해하기 어려움

### 구현
### 상위 클래스
```
public abstract class Smoothie {
    protected String description = "Unknow Smoothie";
    
    public String description() {
        return description;
    }
    
    public abstract int cost();
}
```
### Concrete 클래스
```
public class OrangeSmoothie {
    public OrangeSmoothie() {
        description = "Orange Smoothie";
    }
    
    public int cost() {
        return 5000;
    }
}
```
```
public class MangoSmoothie {
    public MangoSmoothie() {
        description = "Mango Smoothie";
    }
    
    public int cost() {
        return 6000;
    }
}
```

### Condiments 클래스
```
public abstract class CondimentDecorator extends Smoothie {
    protected Smoothie smoothie;
    public abstract String getDescription();
}
```
```
public class MuscleInhancer extends CondimentDecorator {
    public MuscleInhancer(Smoothie smoothie) {
        this.smoothie = smoothie;
    }
    
    public String getDescription() {
        return smoothie.getDescription() + ", muscle inhancer";
    }
    
    public int cost() {
        return 500 + smoothie.cost();
    }
}
```
```
public class SkinInhancer extends CondimentDecorator {
    public SkinInhancer(Smoothie smoothie) {
        this.smoothie = smoothie;
    }
    
    public String getDescription() {
        return smoothie.getDescription() + ", skin inhancer";
    }
    
    public int cost() {
        return smoothie.cost() + 500;
    }
}
```

### Test 코드
```
public class SmoothieKing {
    public static void main(String[] args) {
        Smoothie smoothie = new MangoSmoothie();
        smoothie = new SkinInhancer(smoothie);
        smoothie = new MuscleInhancer(smoothie);
        
        System.out.println(smoothie.getDescription() + " \" + smoothie.cost());
    }
}
```

### 다이어그램
![Untitled Diagram drawio (5)](https://user-images.githubusercontent.com/82352179/148687560-0db5870f-84d8-4ebe-b882-715a57f0646b.png)


## 10. 파사드 Facade
* Structural - Object

### 목적
* 복잡한 소프트웨어 바깥쪽의 코드가 라이브러리 안쪽 코드에 의존하는 일을 감소
* 간단한 인터페이스 제공

### Use When
* Client와 Sub-classes 사이의 통합 인터페이스가 필요할 때

### 구현
### 서브클래스1 (불을 켜고 끔)
```
public class Light {
    private boolean on = false;
    
    public void turnOn() {
        on = true;
        System.out.println("불을 켰다.");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("불을 껐다.");
    }
    
    public boolean isOn() {
        return on;
    }
}
```

### 서브클래스2 (옷을 벗고 입음)
```
public class Clothes {
    private boolean on = false;
    
    public void takeOn {
       on = true;
       System.out.println("옷을 입었다.");
    }
    
    public void takeOff() {
       on = false;
       System.out.println("옷을 벗었다.");
    }
    
    public boolean isOn() {
       return on;
    }
}
```

### 서브클래스3 (티비를 켜고 끔)
```
public class Tv {
    private boolean on = false;
    
    public void turnOn() {
        on = true;
        System.out.println("티비를 켰다.");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("티비를 껐다.");
    }
    
    public boolean isOn() {
        return on;
    }
}
```

### Facade 클래스
```
public class Facade {
    private Light light;
    private Clothes clothes;
    private Tv tv;
    
    public Facade(Light light, Clothes clothes, Tv tv) {
        this.light = light;
        this.clothes = clothes;
        this.tv = tv;
    }
    
    public void in() {
        System.out.println("집에 들어와서 한 일:");
        if(!light.isOn()) {
            light.on();
        }
        
        if(clothes.isOn()) {
            clothes.off();
        }
        
        if(!tv.isOn()) {
            tv.on();
        }
    }
    
    public void out() {
        System.out.println("집을 나올 때 한 일:");
        if(light.isOn()) {
            light.off();
        }
        
        if(!clothes.isOn()) {
            clothes.on();
        }
        
        if(tv.isOn()) {
            tv.off();
        }
    }
}
```

### 다이어그램
![Untitled Diagram drawio (7)](https://user-images.githubusercontent.com/82352179/149653991-57c212b7-c192-4bff-9aab-c64825893d5b.png)


## Reference
https://lktprogrammer.tistory.com/42
https://break-over.tistory.com/47


## 11. 플라이웨이트 Flyweight
* Structural - Object

### 목적
* 공유(Sharing)을 통해 대량의 객체를 효과적으로 지원하기 위해서
* 같은 사항을 여러번 메모리에 올릴 필요가 없기 때문에

### Use When
* 어플리케이션에 의해 생성되는 객체의 수가 많을 때
* 생성된 객체가 오래도록 메모리에 상주하고, 사용되는 횟수가 많을 때
* 객체의 외적 특성이 클라이언트 프로그램으로부터 정의되어야 할 때

### 구현
### Flyweight Interface
```
public interface Shape {
    public void draw();
}
```
### ConcreteFlyweight Class
```
public class Rectangle implements Shape {
    private String innerColor;
    private int yLen;
    private int xLen;
    
    public Rectangle(String color) {
        this.innerColor = color;
    }
    
    public void setInnerColor(String color) {
        this.innerColor = color;
    }
    
    public void setYLen(int yLen) {
        this.yLen = yLen;
    }
    
    public void setXLen(int xLen) {
        this.xLen = xLen;
    }

    @Overrride
    public void draw() {
        System.out.println("Rectangle [color = " + innerColor
                           + ", x length = " + xLen
                           + ", y length = " + yLen);
    }
}
```
### FlyweightFactory Class
```
public class ShapeFactory {
    private static final HashMap<String, Rectangle> rectangleMap = new HashMap<>();
    
    public static Shape getRectangle(String color) {
        Rectangle rectangle = (Rectangle)rectangleMap.get(color);
        
        if(null == rectangle) {
            rectangle = new Rectangle(color);
            rectangleMap.put(color, rectangle);
            System.out.println("[새로운 사각형은 " + color + "색의 사각형입니다.");
        }
        
        return rectangle;
    }
}
```

### 다이어그램
![Untitled Diagram drawio (8)](https://user-images.githubusercontent.com/82352179/149654473-e81bfeb6-add7-476a-8109-d321765d2993.png)

## Reference
https://readystory.tistory.com/137
https://lee1535.tistory.com/106


## 12. 프록시 Proxy
* Structural - Object

### 목적
* 대리자 객체를 통해 메서드를 호출, 반환 값을 받는지 모르게 처리하기 위해
* 실제 기능을 수행하는 객체 대신 가상의 객체를 사용하여 로직의 흐름을 제어할 때

### Use When
* 흐름의 제어는 필요하되, 결과값을 조작하거나 변경하지는 않아야 할 때
* 원래 하려던 기능을 수행하며, 부가적인 작업을 수행하려 할 때
* 비용이 많이 드는 연산을 실제로 필요한 시점에 수행해야 할 때

### 구현
### 인터페이스
```
public interface Order {
    String cookHamburger();
}
```
### 실제 수행 객체 (요리사)
```
public class Cook implements Order {
    @Override
    public String cookHamburger(int num) {
        return num + "번 햄버거를 조리합니다.";
    }
}
```
### Proxy 객체 (키오스크)
```
public class Keyosk implements Order {
    Order order;

    @Override
    public String cookHamburger(int num) {
        System.out.println(num + "번 햄버거 주문을 받았습니다.");
        System.out.println("요리사에게 햄버거 요리를 요청합니다.");
        order = new Cook();
        return cook.cookHamburger(num);
    }
}
```

### 다이어그램
![Untitled Diagram drawio (9)](https://user-images.githubusercontent.com/82352179/150670177-fda642a7-56aa-4492-9be4-1ca833f2132a.png)

### Reference
https://limkydev.tistory.com/79
https://jdm.kr/blog/235


## 13. 책임 연쇄 Chain of Responsibility
* Behavioral

### 목적
* 클라이언트의 요청을 처리할 수 있는 처리 객체를 집합으로 만들어 결합을 느슨히 하기 위해서
* 처리 객체의 추가, 삭제를 유연성있게 하기 위해서

### Use When
* 요청의 발신자와 수신자를 분리할 때
* 요청을 처리할 수 있는 객체가 여러개일 때 그 중 하나에 요청을 보내려는 경우
* 코드에서 처리 객체를 명시적으로 지정하고 싶지 않은 경우

### 구현
### Handler
```
interface BlockToy {
   void setNextBlock(BlockToy nextBlock);
   void build(Block block);
}
```

### Object
```
class Block {
   private int number;
   
   public Block(int number) {
       this.number = number;
   }
   
   public String getNumber() {
       return this.number;
   }
}
```

### Concrete Handler
```
class RedBlock implements BlockToy {
    private BlockToy chain;
    
    @Override
    public void setNextBlock(BlockToy nextBlock) {
        this.chain = nextBlock;
    }
    
    @Override
    public void buildBlock(Block block) {
        if(block.getNumber == cur) {
            int number = block.number();
            System.out.println("This block is " + number + " block.");
            if(3 != cur) {
                cur++;
                this.chain.buildBlock(new Block(cur));
            }
        } else {
            this.chain.buildBlock(block);
        }
    }
}
```

### 다이어그램
![Untitled Diagram drawio (10)](https://user-images.githubusercontent.com/82352179/151704687-aa01c0af-7ab0-406b-84bc-9a77439027cb.png)

### Reference
https://always-intern.tistory.com/1

## 14. 커맨드 Command
* Behavioral - Object

### 목적
* 실행할 기능을 캡슐화하여 재사용성이 높은 클래스를 설계하기 위해서
* 실행을 요구하는 호출자 클래스와 실제 기능을 실행하는 수신자 클래스 사이의 의존성을 제거하기 위해서

### Use When
* 이벤트가 발생했을 때 실행할 기능이 다양한 경우
* 이벤트에 변경이 필요할 때, 이벤트를 발생시키는 클래스를 변경하지 않고 재사용하고자 하는 경우

### 구현
### Command
```
public inter face Command {
   public abstract void execute();
}
```
### Walldoor
```
public class Walldoor {
    private Command command;
    public Walldoor(Command command) {
        setCommand(command);
    }
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressed() {
        command.execute();
    }
}
```

### DoorCommand
```
public class Door {
   public void open() { System.out.println("Open the door"); }
}
```
```
public class DoorCommand implements Command {
    private Door door;
    public DoorCommand(Door door) { this.door = door; }
    public void execute() { door.open(); }
}
```

### TempCommand
```
public class Temp {
   public void turnOff() { System.out.println("Turn off the temp."); }
}
```
```
public class TempCommand implements Command {
    private Temp temp;
    public TempCommand(Temp temp) { this.temp = temp; }
    public void execute() { temp.turnOff(); }
}
```

### 다이어그램
![Untitled Diagram drawio (11)](https://user-images.githubusercontent.com/82352179/151704977-f3275100-495c-4b22-91a4-21ee525fab85.png)

### Reference
https://gmlwjd9405.github.io/2018/07/07/command-pattern.html
