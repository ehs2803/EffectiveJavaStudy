# 상속보다는 컴포지션을 사용하라

상속은 코드 재사용을 구현하기 위한 강력한 방법이다. 

하지만 잘못 사용할 경우 오류를 내기 쉬운 프로그램을 만들게 된다. 이러한 문제는 상위 클래스와 하위 클래스를 동일한 개발자가 개발하지 않은 경우에 발생하게 된다. 따라서 다른 클라이언트가 내가 만든 클래스를 상속받을 수 있게 하려면 주의해야 한다.

상속은 코드 재사용성을 높여주지만 캡슐화를 깨뜨리게 된다. 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다는 말이다. (상위 클래스가 변경되는 등의 변화로 인해서!!)

따라서 상위 클래스 개발자가 상속을 고려한 문서화를 하거나 상속하려는 클라이언트가 상위 클래스의 변화에 맞춰 하위 클래스를 수정해줘야 한다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    ...
    @Override
    public boolean add(E e) {
    	this.addCount++;
        return super.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
    	this.addCount += c.size();
        return super.addAll(c);
    }
    ...
}
```
이미 구현되어 있는 Collection 패키지의 HashSet을 상속받는 클래스를 만든다고 가정해보자. Set에 추가한 Element의 수를 파악하기 위해서 인스턴스 변수 addCount를 사용했다.

하지만, addAll을 통해 [1, 2, 3] 을 추가했을 때 addCount 는 6이라는 의도하지 않은 결과를 가져온다. 왜냐하면 사실 HashSet 의 addAll 은 내부적으로 add 를 호출하기 때문이다. 하지만 우리는 이러한 정보를 알 수가 없다. 이렇게 내부구현방식을 모를 때 심각한 문제가 발생할 수 있다.

이러한 문제 외에도 상위 클래스에 새로운 메서드가 추가되었을 때와 같은 상황에서 다양한 문제가 발생할 수 있다. 


# Composition

Composition은 Has-a 관계를 구현하기 위한 설계 기술이다. 참고로 상속은 Is-a 관계를 구현한다. 이러한 컴포지션은 상속을 대신해 코드 재사용의 목적으로 사용될 수 있다.

이러한 컴포지션은 다른 클래스의 객체를 참조하는 인스턴스 변수를 사용해 구성할 수 있다.

![image](https://user-images.githubusercontent.com/65898555/211176799-67867652-2baa-4dda-9f24-c25ebc287ff6.png)

위 이미지를 보면 AccountContext 클래스는 Account 클래스를 상속받지 않고 인스턴스 변수로써 포함했다. 이러한 구현방식을 컴포지션이라고 볼 수 있다.

물론 이러한 방식 외에도 다양한 방식으로 활용될 수 있다. 

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }
    
    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o); }
    ...
}

public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> s) { super(s); }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
    	addCount += c.size();
        return super.addAll();
    }
}
```
Set<E> 인터페이스를 구현한건 단순히 명세를 따르기 위함이다. 굳이 구현하지 않고 메소드 이름을 자기 마음대로 구현해도 상관없다. 
(이렇게 구현하는 것을 래퍼 클래스라고 한다. 래퍼 클래스로 구현이 가능하다면 하는 것이 유연하고 견고하다고 한다)


인스턴스 변수로 Set<E> 의 참조를 두고 내부 기능과 매핑되는 메서드들을 작성하는 것이다. 이러한 방식을 Forwording 이라고 한다. 


# 장점
  
- 상속으로 구현하게 되면 상위 클래스의 모든 public 메서드가 클라이언트에게 공개된다. 하지만, 컴포지션을 사용해 구현하게되면 개발자가 원하는 메서드만 클라이언트에게 공개할 수 있다.
- 상위 클래스의 내부구현을 숨길 수 있다.
- JAVA 에서 지원하지 않는 다중상속의 목적을 달성할 수 있다.
- 상위 클래스에서 제공하는 메서드를 더 나은 버전으로 개선할 수 있다. (충분히 유연하다) Override 는 아니지만 동일한 기능을 제공할 수 있다.
- 참조하고 있는 인스턴스 변수를 변경해 프로그램을 동적으로 변경할 수 있다.
- 상위 클래스의 메서드 형태와 관계없이 유연하게 하위 클래스의 메서드를 정의할 수 있다. 상위 클래스에서는 String 반환 값을 가지지만 하위 클래스에서는 Integer 를 가지도록 할 수 있다.

  
컴포지션을 써야하는 상황에서 상속을 잘못 사용한 케이스가 자바 플랫폼 라이브러리에도 존재한다. 예를들어 Stack 은 Vector 가 아니기 때문에 상속했으면 안됐다. 또한 Properties 도 HashTable 을 상속해서는 안됐다.

Stack 이 Vector 를 상속한 탓에 Stack 클래스에는 push, pop 메서드와 get, set 이 모두 존재하게 되어버렸다. 컴포지션을 사용했더라면 이러한 문제가 발생하지 않았을 것이다.

반드시 상속을 사용해야겠다면 다음과 같은 문제에 대해서 고민해보자.

첫 번째로 상위 클래스와 하위 클래스가 Is-a 관계가 완벽하게 성립하는지 확인해보자. 두 번째로 상속받고자 하는 클래스의 API 에 결함이 없는지 확인하고 이 결함이 전파되어도 되는지 확인해보자. (상속은 이러한 결함까지도 클라이언트에게 공개하기 때문이다)
