# 배열보다는 리스트를 사용하라

### 배열과 리스트 차이점

1. 첫 번째 - 배열은 공변인 반면 리스트는 불공변이다.

배열의 경우 Sub가 Super의 하위 타입이라면 Sub[]는 배열 Super[]의 하위 타입이 된다.

반면, 리스트의 경우 서로 다른 타입 Type1, Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.
  
  
[문법상 허용 - 런타임 실패]
  
```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException 발생
```
  
[문법상 불허용 - 컴파일 실패]
  
```java
List<Object> objectList = new ArrayList<>(); // 호환되지 않는 타입이다.
objectList.add("타입이 달라 넣을 수 없다.");
```
배열은 이런 오류 발생을 런타임시에 알 수 있지만 리스트를 사용하면 컴파일시에 바로 알 수 있다.
  
  
2. 두 번째 - 배열은 실체화된다.
  
배열은 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 Long 타입 배열에 String 타입 데이터를 입력하려고하면 ArrayStoreException이 발생한다.

반면, 리스트는 타입 정보가 런타임에는 소거된다. 원소 타입을 컴파일시에만 검사하며 런타임에는 알 수 없다는 말이다.

이런 이유로 배열은 제네릭 타입(new List<E>[]), 매개변수화 타입(new List<String>[]), 타입 매개변수(new E[]) 로 사용할 수 없다. 이런 식으로 코드를 작성하려하면 제네릭 배열 생성 오류를 일으킨다.
  
제네릭 배열을 만들지 못하게하는 이유는 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있기 때문에 타입 안전하지 않기 때문이다. 또한 런타임에 ClassCastException이 발생하는 것을 막아주겠다는 제네릭 타입 시스템 취지에 어긋나는 일이기도 하다.

따라서 이런일을 방지하기위해 (1)번 과정에서 컴파일오류를 발생시킨다.
  
### 실체화 불가 타입
  
E, List<E>, List<String> 같은 타입을 실체화 불가 타입이라고 한다.

쉽게 말해, 실체화 되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다. 소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?>, Map<?,?> 같은 비한정적 와일드카드 타입뿐이다.

참고로, 배열은 비한정적 와일드카드로 만들수는 있지만 유용하게 쓰일 일은 거의 없다.
  
### @SafeVarargs
  
@SafeVarargs는 메서드 작성자가 해당 메서드가 타입 안전하다는 것을 보장하는 장치이다.
  
제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다. 또한 제네릭 타입과 가변인수 메서드를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다.

가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생한다.

이 때 @SafeVarargs를 사용하면 잠재적 오류에 대한 경고를 무시함으로써 해결할 수 있다. 만약, 메서드가 타입 안전하지 않다면 절대 @SafeVarargs를 사용해서는 안된다.
  
[@SafeVarargs - 잠재적 오류 경고 무시]
  
```java
public class SafeVars {
    @SafeVarargs
    public static void print(List... names) {
        for (List<String> name : names) {
            System.out.println(name);
        }
    }

    public static void main(String[] args) {
        SafeVars safeVars = new SafeVars();
        List<String> list = new ArrayList<>();

        list.add("b");
        list.add("c");
        list.add("a");
        print(list);
    }
}
```

### 배열대신 컬렉션을 사용하자.
  
배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분 배열 E[]대신 List<E>를 사용하면 해결된다.

조금 복잡해지고 성능이 나빠질 수 있지만 타입안정성이 보장되고 상호 운용성이 좋아진다.
  
[Chooser - 제네릭 적용 필요]
  
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(final Object[] choiceArray) {
        this.choiceArray = choiceArray;
    }

    public Object choose(){
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```
위 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 만약 타입이 다른 원소가 들어있으면 런타임시에 형변환 오류가 발생한다.
  
[리스트 기반 Chooser - 타입 안정성 확보]
  
```java
public class ListChooser {
    private final List<T> choiceList;

    public ListChooser(final Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }
    
    public T choose(){
        Random random = ThreadLocalRandom.current();
        return choiceList[random.nextInt(choiceList.size())];
    }
}
```
코드는 조금 길어졌지만 리스트를 사용함으로써 런타임에 ClassCastException을 만날일이 없어졌다.
  
### 정리

배열은 런타임에는 타입 안전하지만 컴파일시에는 그렇지 않다. 제네릭은 배열과 반대이다. 되도록이면 제네릭을 사용하고 둘을 섞어 사용하다 경고를 만날경우 배열을 리스트로 대체하자.
