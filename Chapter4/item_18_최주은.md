# Item 18. 상속보다는 컴포지션을 사용하라
[블로그 링크](https://jueun275.tistory.com/entry/Item-18-%EC%83%81%EC%86%8D%EB%B3%B4%EB%8B%A4%EB%8A%94-%EC%BB%B4%ED%8F%AC%EC%A7%80%EC%85%98%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC)

<br/>

> **"메서드의 호출과 달리 상속은 캡슐화를 깨뜨린다."**

이 장에서의 상속은 **클래스가 다른 클래스를 확장하는 구현 상속**을 말하는 것이며, 클래스가 인터페이스를 구현하거나 인터페이스가 다른 인터페이스를 확장하는 인터페이스 상속에 대한 이야기는 아니다.

<br/>
<br/>

---

<br/>

## 상속이란?

**상속(inheritance)은**

-   객체 지향 프로그래밍(OOP)에서, 객체들 간의 관계를 구축하는 방법
-   기존 클래스를 확장하여 새 클래스를 만드는 것.
-   이때 기존 클래스는 상위(부모) 클래스 확장된 클래스는 하위(자식) 클래스라고 하며
-   하위 클래스는 상위 클래스의 종류로 상위 클래스가 가진 특성을 재사용한다.
-   하위 클래스에서 extends라는 키워드를 통해 상위 클래스를 상속할 수 있다.

```java
class Sub extends Super {}
```
<br/>
<br/>

## 상속의 단점

#### **상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.**

상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있습니다.

**상속을 잘못 사용한 예제코드**

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    public InstrumentedHashSet(){}
    
    public InstrumentedHashSet(int initCap, float loadFactor){
    	super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

이 클래스는 잘 구현된 것처럼 보이지만 제대로 작동하지 않는다.

<br/>

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

위와 같이 InstrumentedHashSet 클래스의 인스턴스에 addAll 매소드로  원소 3개를 더한 다음 getAddCount를 호출하면 3을 반환하리라 기대하겠지만 실제로는 6을 반환한다. 왜 이런 문제가 발생한 것일까? 원인은 HashSet의 addAll 메서드가 add메서드를 사용해 구현된 데 있다.

<center>
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fm5L26%2FbtrVQO46nae%2FbhH6WEckgk9VfKJkxynY8K%2Fimg.png"
    width="80%"
    height="80%"
  />
</center>

</br>

위와 같이 HashSet의 addAll은 add 메서를 호출해서 각 원소를 추가하는데, 이때 불리는 add는 InstrumentedHashSet에서 재정의한 메서드다. 따라서 addCount에 값이 중복해서 더해져, 최종값이 6이 된 것이다.

</br>

#### **하위 클래스에서 addAll 메서드를 재정의하지 않으면?**

당장은 제대로 동작할지 모르나, Hashset의 addAll메서드가 add메서드를 이용해 구현했음을 가정한 해법이라는 한계가 있다. 이처럼 자신의 다른 부분을 사용하는 '자기 사용(self-use)'자기 사용(self-use) 여부는 해당 클래스의 내부 구현 방식에 해당하며 다음 릴리즈에도 유지될지는 알 수 없다.

</br>

####  **addAll 메서드를 다른 식으로 재정의 한다면?**

이 경우 HashSet의 addAll을  호출하지 않기 때문에 addAll이 add를 사용하는지와 상관없이 결과가 옳다는 점에서 조금 더 나은 방법이다. 그러나 상위 클래스의 메서드 동작을 다시 구현하는 방식은 어렵고, 시간도 더 소요될뿐더러 자칫 오류를 내거나 성능을 저하시킬 수 있다.

</br>

#### **메서드를 재정의 하지 않고 새로운 메서드를 추가한다면?**

메서드를 재정의 하는 대신 새로운 메서드를 추가한다고 해도 다음 릴리스에서 상위 클래스에 새로운 메서드가 추가될 경우,  새롭게 추가된 메서드를 통해서 허용되지 않은 동작을 수행할 수 있게 될 수도 있다. 만약 하위 클래스에 추가한 메서드와 상위 클래스에 새롭게 추가된 메서드의 시그니처가 같고 반환 타입이 다르다면 컴파일조차 되지 않는다.

</br>

#### **그래서 상속은 쓰면 안 되는 것인가..?**

상속이 적절하게 사용되면 뒤에서 알아볼 컴포지션보다 강력하고, 개발하기도 편리하다. 단, 상속이 적절하게 사용되려면 최소 다음과 같은 조건을 만족해야 한다.

1.  확장을 고려하고 설계한 확실한 is - a 관계일 때
2.  API에 아무런 결함이 없는 경우, 결함이 있다면 하위 클래스까지 전파돼도 괜찮은 경우

</br>
</br>

## 자바 플랫폼 라이브러리 에서의 위반

자바 플랫폼 라이부로리에서도 이 is-a 관계에서만 상속을 사용해야 한다는 원칙을 위반한 클래스들이 있다

-   스택은 벡터가 아닌데 벡터를 상속받았고
-   properties는 HashTable이 아닌데 HashTable을 상속받았다

</br>

<center>
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbSmQiv%2FbtrVZOwuIGj%2FAF2yFB1SIbgi6pVU4ahtc0%2Fimg.png"
    width="100%"
    height="100%"
  />
</center>

</br>

<center>
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwzbBW%2FbtrV0iDSvfK%2FiltaESwEBCoxHfCt71sNUk%2Fimg.png"
    width="100%"
    height="100%"
  />
</center>

</br>
</br>

### **properties에 어떤 문제점들이 생겼는가?**

#### **1\. 사용자를 혼란스럽게 할 수 있다.**

```java
// HashTable을 상속하는 Properties
Properties p = new Properties();

p.get(key); 
p.getProperty(key);
```

get, getProperty는 각각 상위 클래스와 구현 클래스에 있는 메서드이다. 그런데 p.get(key), p.getProperty(key)는 결과가 다를 수 있다. 전자가 Properties의 기본 동작인 데 반해, 후자는 Properties의 상위 클래스인 HashTable로부터 물려받은 메서드이기 때문이다.

</br>

#### **2\. 하위클래스의 불변식을 깨버릴 수 있다.** 

Properties는 키와 값으로 문자열만 허용하도록 설계하려 했으나, 상위클래스인 HashTable의 메서드를 직접 호출하는 경우 이 불변식을 깨버릴 수 있다. 불변식이 한번 깨지면 load, store 같은 다른 Propertie API는 더 이상 사용할 수 없다. 이 문제가 밝혀졌을 때 이미 수많은 사용자가 Propertiey의 키나 값으로 문자열 이외의 타입을 사용하고 있었다.

</br>

스택과, properties 모두 컴포지션을 사용했다면 더 좋았을 것이다.

</br>
</br>

## 컴포지션 이란

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고, private 필드로 기존 클래스의 인스턴스를 참조하게 하도록 설계하는 방법이다. 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.

</br>

**컴포지션(composition)** : 기존 클래스가 새로운 클래스의 구성요소로 쓰인다.(private 필드)  
**전달(forwarding)** : 새 클래스의 인스턴스 메서드들이 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환하는 방식  
**전달 메서드(forwarding method)** : 새 클래스의 메서드들

</br>
</br>

## 컴포지션을 사용하는 이유

위와 같은 컴포지션을 사용하면 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않는다.

<br/>

**컴포지션 예제코드**

-   재사용할 수 있는 전달 클래스 

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

-   레퍼 클래스 - 상속 대신 컴포지션 사용

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount(){ return addCount; }
}
```

-   InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
-   임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.

```java
InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
s.addAll(List.of("틱", "탁탁", "펑"));
```

다시 위의 상황에서 addAll을 호출하고 getAddCount를 호출하면 원하는 값인 3이 잘 나온다.

상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 하지만 컴포지션 방식은 한 번만 구현해 두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp)); 
Set<E> s = new Instrumented<>(new HashSet<>(INIT_CAPACITY));
```

```java
static void walk(Set<Dog> dogs) {
	// 기존 Set을 덮어씌워 사용 -> 작동은 똑같다.
	InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
	....
}
```

다른 Set인스턴스를 감싸고(wrap) 있다는 뜻에서 **InstrumentedSet 같은 클래스를  래퍼 클래스(wrapper class)라고 하며,** 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다. 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다. 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.

</br>
</br>

## 래퍼 클래스의 단점

래퍼 클래스는 단점이 거의 없다. 

래퍼 클래스가 콜백(callback) 프레임워크와는 어울리지 않는다는 점만 주의하도록 하자.

</br>

**래퍼 클래스와 SELF 문제**  
콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다.  
내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 되는데, 이를 SELF 문제라고 한다.

</br>
</br>

## 핵심정리

-   상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
-   상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다.
    -   is-a 관계일 때도 안심할 수만은 없다.
    -   하위 클래스의 패키지가 상위 클래스와 다르다.
    -   상위 클래스가 확장을 고려해 설계되지 않았을 수도 있기 때문이다.
-   상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용한다.
    -   특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다.
    -   래퍼 클래스는 하위 클래스보다 견고하고 강력하다.