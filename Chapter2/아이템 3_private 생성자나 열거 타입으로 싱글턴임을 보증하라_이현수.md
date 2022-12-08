# 아이템3 : private 생성자나 열거 타입으로 싱글턴임을 보증하라

# 싱글톤(singleton)

: 인스턴스를 오직 하나만 생성할 수 있는 클래스
ex) 함수(무상태 객체), 설계상 유일해야 하는 시스템 컴포넌트

생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해 둔다.

# 1. public static 멤버가 final 필드인 방식
public static final 필드 방식의 싱글턴
```java
package item3.field;

public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
private 생성자는 public static final필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.
public이나 protected 생성자가 없으므로 Elvis클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

### 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
- 간결하다.

### 단점

```java
Constructor<Elvis> constructor = (Constructor<Elvis>) test1.getClass().getDeclaredConstructor();
constructor.setAccessible(true);

Elvis test2 = constructor.newInstance();
assertNotSame(test1, test2);
```


- 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
- 리플렉션 API: java.lang.reflect, class 객체가 주어지면, 해당 클래스의 인스턴스를 생성하거나 메소드를 호출하거나, 필드에 접근할 수 있다.

```java
private Elvis(){
    if(INSTANCE != null){
        throw new RuntimeException("생성자 호출 불가");
    }
}
```
생성자를 수정해 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

# 2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
정적 팩터리 방식의 싱글턴
```java
package item3.staticfactory;

public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }
}
```
Elvis.getInstance()는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis인스턴스는 만들어지지 않는다.
(리플렉션을 통한 예외는 똑같이 적용된다.)

### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. (getInstance()호출부 수정 없이 내부에서 private static 이 아닌 새 인스턴스를 생성해주면 된다) 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드 별로 다른 인스턴스를 넘겨주게 할 수 있다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.

```java
Supplier<Elvis> elvisSupplier = Elvis::getInstance;
Elvis elvis = elvisSupplier.get();
```
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다. Supplier: get메서드 만을 가지고 아무 type이나 리턴할 수 있는 인터페이스.

이러한 장점을 필요하지 않다면 public 필드 방식이 좋다.

# 두 방식의 문제점

각 클래스를 직렬화한 후 역직렬화할 때 새로운 인스턴스를 만들어서 반환한다.

역직렬화는 기본 생성자를 호출하지 않고 값을 복사해서 새로운 인스턴스를 반환한다. 그때 통하는게 readResolve() 메서드이다.

이를 방지하기 위해 readResolve 에서 싱글턴 인스턴스를 반환하고, 모든 필드에 transient(직렬화 제외) 키워드를 넣는다.

```java
public class Singleton implements Serializable {
    public static Singleton INSTANCE = new Singleton();
    private int value;
    
    private Singleton(){};
    
    protected Object readResolve() {
        return INSTANCE;
    }
    
    public void setValue(int value){
        this.value = value;
    }
    public int getValue(){
        return this.value;
    }
}
```
싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현하고 선언하는 것만으로 부족하다. 모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야 한다.
역직렬화 시 반드시 호출되는 readResolve 메서드를 싱글턴을 리턴하도록 수정.

이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.


# 3. 원소가 하나인 열거 타입을 선언하는 방식

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

```java
package item3.enumtype;

public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
1. 상수들만 모아놓은 클래스라고 할 수 있다. 때문에 클래스처럼 메소드, 생성자를 모두 가질 수 있다.
2. Private 생성자를 갖는다

```java
public enum SingletonEnum {
    INSTANCE;
    int value;
    
    public int getValue() {
        return value;
    }
    public void setValue(int value) {
        this.value = value;
    }
}
public class EnumDemo {
    
    public static void main(String[] args) {
        SingletonEnum singleton = SingletonEnum.INSTANCE;
        
        System.out.println(singleton.getValue());
        singleton.setValue(2);
        System.out.println(singleton.getValue());
    }
}
```

Enum은 private 생성자로 인스턴스 생성을 제어하며, 상수만 갖는 특별한 클래스이기 때문에 싱글톤의 성질을 일반적으로 갖게된다. 때문에 싱글톤을 간단히 구현할 때 사용되곤 한다.

복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.

단, 만들려는 싱글턴이 Enum 이외의 다른 상위 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.
