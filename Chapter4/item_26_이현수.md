# 로 타입은 사용하지 말라

### 로(raw) 타입이란?


로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않은 때를 말한다. 예를 들어서 List<E>의 로 타입은 List다. 제네릭 도입 이후에도 로 타입을 지원하는 이유는 기존 자바 버전과의 호환성 때문이다. 

 

아래 컬렉션 예제 코드에서처럼 로 타입을 사용하여 객체를 저장 하면 컴파일 할 때는 에러가 발생하지 않는다. 그리고 런타임 때 저장된 객체를 꺼내면서 오류가 발생한다. 오류는 컴파일 시점에 발견하는 것이 좋다.
  
  
```java
//로 타입으로 인스턴스 저장
private final Collection stamps = ...;
stamps.add(new Coin());

//데이터 꺼낼 때 오류 발생
for(Iterator i = stamps.iterator(); i.hasNext();){
    Stamp stamp = (Stamp) i.next();  //ClassCaseException 발생
    stamp.cancel();
}
```
  
### List<Object>

  ###

List의 로 타입은 권장하지 않지만 List<Object> 처럼 사용하는 것은 괜찮다. 왜냐하면 모든 타입을 허용한다는 의사를 컴파일러에게 명확하게 전달한 것이기 때문이다. List와 List<Object>의 차이는 List는 제네릭 타입과 무관하다. 

아래 코드의 경우에서는 List를 사용해서 unsafeAdd 메소드를 실행한다. 컴파일 에러가 발생하지 않으며 코드를 실행해야 string.get(0)을 String으로 바꾸는 코드에서 ClassCastException이 발생한다.
  
```java
import java.util.ArrayList;
import java.util.List;

public class UnsafeAdd {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);
    }

    // 로 타입
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }

}
```
  
이번에는 List<Object>로 unsafeAdd 메소드를 작성해보면 컴파일 시점에 오류가 나는 것을 확인할 수 있다. List<String>은 로 타입인 List의 하위 타입이지만 List<Object>의 하위 타입은 아니기 때문이다. 따라서 List<Object>를 사용하면 좀 더 type safe하게 코딩할 수 있다.
  
```java
import java.util.ArrayList;
import java.util.List;

public class UnsafeAdd {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);
    }

    // List<Object>
    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }

}
```

### 원소를 모르는 로 타입 사용
  
원소의 타입을 모르는 로 타입을 사용하고 싶을 수 있다. 다음은 제네릭을 처음 접하는 사람이 작성할만한 코드이다. 이 코드는 동작은 하지만 로 타입을 사용하고 있어서 안전하지 않다.

```java
static int numElementsInCommon(Set s1, Set s2){
    int result = 0;
    for(Object o1 : s1){
        if(s2.contains(o1))
            result++;
    }
    
    return result;
}
```
비한장적 와일드 카드 타입을 대신 사용하는게 좋다. 제네릭 타입을 사용하고 싶지만 실제 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?)를 사용하자.  비한정적 와일드카드 타입을을 사용하면 Collection<?>에는 null외에는 어떤 원소도 넣을 수 없다. 다른 원소를 넣으려하면 컴파일할 때 오류가 나타난다.

### 예외 케이스

로 타입을 쓰지 말라는 규칙에도 소소한 예외가 몇개 있다. class 리터럴에는 로 타입을 사용해야한다. List.class, String[].class, int.class는 허용되지만, List<String>.class, List<?>.class는 허용되지 않는다.

 

두 번째 예외는 instanceof 연산자와 관려이 있다. 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다. 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작한다.
  
```java
// o의 타입이 Set인지 확인한 다음, 와일드카드 타입으로 형변환
// 여기서 로 타입인 Set이 아닌 와일드카드 타입으로 변환함
if( o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```
  
  
  
