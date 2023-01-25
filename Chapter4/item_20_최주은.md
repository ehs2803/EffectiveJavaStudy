# Item 20. 추상 클래스보다는 인터페이스를 우선하라

> 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격구현을 함계 제공하는 방법을 꼭 고려해보자

<br/>

이번 아이템에서는 추상클래스와 인터페이스, 인터페이스의 장점과 골격 구현 클레스에 대해서 다룹니다

<br/>

---

<br/>

## 추상 클래스와 인터페이스

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상클래스, 이렇게 두 가지다.

자바 8부터 인터페이스도  디폴트 메서드를 제공할 수 있게 되어, 이제는 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

<br/>

#### **추상클래스란**

자바에서는 하나 이상의 추상 메서드를 포함하는 클래스를 가리켜 추상 클래스라 한다.

```java
abstract class 클래스이름 {
	pulic abstract 반환타입 메소드이름();   // 추상 메서드
}
```

-   클래스 또는 메서드 앞에 abstract를 붙여 추상클래스, 추상메서드를 만든다.
-   추상 메서드가 하나 이상 존재하면 abstract class로 명시해야 하지만, 추상 클래스가 반드시 추상 메서드를 가질 필요는 없다.
-   생성자를 가질 수 있다

<br/>

**추상메서드**

-   선언부만 있고 구현부가 없다는 의미로 선언부 끝에 바로 세미콜론;을 추가한다.
-   구현부가 없기 때문에 자식  반드시 클래스에서 오버라이딩 해야만 사용할 수 있다.

<br/>

#### **인터페이스란**

모든 메서드가 추상 메서드인 경우

```java
interface class 인터페이스이름 {
	public static final 상수이름 = 값;   // 상수
    
	pulic abstract 반환타입 메소드이름();   // 추상 메서드
    	pulic abstract 반환타입 메소드이름();   // 추상 메서드
}
```

-   상수(static final)와 추상 메서드(abstract method)의 집합이다.
-   생성자를 가질 수 없기 때문에 객체화가 불가능하다.

<br/>

#### **인터페이스와 추상클래스의 가장 큰 차이점**

-   추상클래스: 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.(상속 extends 만 가능)
-   인터페이스: 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

<br/>
<br/>

## 인터페이스의 장점

#### **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다**

요구하는 메서드를 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다.

```java
public class Student extends people implements Comparable {

  @Override
  public int compareTo(Object o) {
    return 0;
  }
}
```

<br/>


#### **믹스인을 정의하는 데에 안성맞춤이다**

클래스는 단일 상속만 가능하고, 클래스 계층구조에는 믹스인을 삽입하기에 합리적인 위치가 없기 때문에 추상 클래스로는 믹스인을 정의할 수 없다.

> 믹스인 이란  
> 믹스인이란 클래스가 구현할 수 있는 타입으로, 대상 타입의 주된 기능에 선택적 기능을 혼합(mixed in) 한다는 의미이다

믹스인 인터페이스엔 대표적으로 Comparable, Cloneable, Serializable 이 존재한다. 

ex)

```java
//Comparable, Iterable 혼합
class TestClass implements Comparable, Iterable {
 . . .
}
```

<br/>

#### **계층구조가 없는 프레임워크를 만들 수 있다.**

타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있다. 하지만 현실에는 계층을 엄격히 구분하기 힘든 개념도 있다. 

<br/>

```java
public interface Singer {
    abstract void sing(String s);
}

public interface SongWriter {
    abstract void compose(int chartPosition);
}
```


가수와 작곡가 싱어송라이터가 있을 때  인터페이스로 정의하면 

<br/>

```java
public class People implements Singer, SongWriter {
    @Override
    public void Sing(String s) {

    }
    @Override
    public void Compose(int chartPosition) {

    }
}
```

 클래스가 Singer와 Songwriter 모두를 구현해도 전혀 문제 되지 않는다

<br/>

```java
public interface SingerSongwriter extends Singer, Songwriter {
    void strum();
    void actSensitive();
}
```

심지어 Singer와 Songwriter 모두를 확장하고 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있다.

<br/>

만약 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야 하므로.(ex 작곡 가능한 가수, 작곡 불가능한 가수)  엄청나게 거대한 계층구조가 만들어진다. 속성이 n개라면 지원해야 할 조합의 수는 2^n 개나 될 것이다

<br/>
<br/>

#### **래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.**

타입을 추상클래스로 정의해 두면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다

<br/>

#### **defalut 메서드**

인터페이스의 메서드 중 구현 방법이 명확한 것이 있다면, 디폴트 메서드로 제공해 줄 수 있다. 이 기법의 예는 21-1의 removeIf 메서드를 보면 된다. 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화해야 한다.

```java
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

<br/>

**디폴트 메서드 제약**

 equals와 hashcode 같은 Object 메서드를 디폴트 메서드로 제공해서는 안된다. 또한 인터페이스는 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다.(단, private 정적 메서드는 예외다.) 마지막으로, 여러분이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

<br/>
<br/>

## 템플릿 메서드 패턴

인터페이스와 추상 골격 구현 클래스(skeletal implementation)를 함께 제공하는 방식으로 인터페이스와 추상클래스의 장점을 모두 취하는 방법이다. (즉 인터페이스 + 추상클래스)

<br/>

인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 그러고 골격 구현 클래스 나머지 메서드들까지 구현한다.

<br/>

좋은 예로, 컬렉션 프레임워크의 AbstractCollection, AbstractSet, AbstractList, AbstractMap 각각이 핵심 컬렉션 인터페이스의 골격 구현이다.

<center class="half">
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8QbhT%2FbtrW9oXY9rP%2FE1ryRdJB0UpPoidaCGndW1%2Fimg.png"
    width="100%"
    height="100%"
  />
</center>

<br/>

골격 구현클래스를 사용해 완성한 구체 클래스

```java
static List<Integer> intArrayAsList(int[] a)  {
    Objects.requireNonNull(a);

    //다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
    //더 낮은 버전을 사용한다면 <Integer>로 수정하자.
    return new AbstractList<>()  {
      @Override
      public Integer get(int i)  {
        return a[i]; //오토박싱
      }

      @Override
      public Integer set(int i, Integer val)  {
        int oldVal = a[i];
        a[i] = val; //오토언박싱
        return oldVal; //오토박싱
      }

      @Override
      public int size()  {
        return a.length;
      }
    };
  }
```

완벽히 동작하는 List구현체를 반환하는 정적 팩터리 메서드로 AbstractList 골격 구현을 활용했다. List 구현체가 제공하는 기능들을 생각하면, 이 코드는 골격 구현의 힘을 잘 보여주는 인상적인 예라 할 수 있다. int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터이기도 하다.

<br/>

### **골격 구현클래스**

추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다. 골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝나지만, 꼭 이렇게 해야 하는 것은 아니다. 구조상 골격 구현을 확장하지 못하는 처지라면 인터페이스를 직접 구현해야 한다. 이런 경우라도 인터페이스가 직접 제공하는 디폴트 메서드의 이점을 여전히 누릴 수 있다.

<br/>

또한 골격 구현 클래스를 우회적으로 이용할 수도 있다. 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

<br/>

아이템 18에서 다룬 래퍼 클래스와 비슷한 이 방식을 시뮬레이트한 다중 상속이라 하며, 다중 상속의 많은 장점을 제공하는 동시에 단점은 피하게 해 준다.

<br/>
<br/>

## 골격구현 클래스 작성방법

1\. 인터페이스를 확인해 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다. 이 기반 메서드들은 골격 구현에서는 추상 메서드들이 될 것이다.

<br/>

2\. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다.

단 equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공하면 안 되다.

<br/>

3. 만약 인터페이스의 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 이유는 없다.

<br/>

4\. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면. 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 추가해도 된다. 이때 골격 구현 클래스는 abstract 클래스가 된다.

<br/>

간단한 예로 Map.Entry 인터페이스를 살펴보자(Entry는 Map의 내부 인터페이스)

```java
 interface Entry<K, V> {

        K getKey();  //기반 메서드
        
        V getValue(); //기반 메서드
    
        V setValue(V value);
      
        boolean equals(Object o);
        
        int hashCode();
       
	// 디폴트 메서드 생략

    }
```

getKey, getValue는 확실히 기반메서드이고 선택적으로 setValue를 포함할 수 있다. 이 인터페이스는 equals, hashCode의 동작방식도 정의해 놨다.
<br/>
Object메서드들은 디폴트 메서드로 제공해서는 안 되므로, 해당 메서드들은 모두 골격 구현 클래스에 구현한다. toString도 기반 메서드를 사용해 구현해 놨다.

<br/>

골격 구현 클래스

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
	
    //변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override 
    public V setValue(V value)  {
    	throw new UnsupportedOperationException();
    }
    
    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override
    public boolean equals(Object o)  {
    	if(o == this)
        	return false;
        if(!(o instanceof Map.Entry))
        	return false;
        Map.Entry<?, ?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey())
        	&& Objects.equals(e.getValue(), getValue());
    }
    
    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override 
    public int hashCode()  {
    	return Objects.hashCode(getKey())
        	^ Objects.hashCode(getValue());
    }
    
    @Override
    public String toString()  {
    	return getKey() + '=' + getValue();
    }
}
```

Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 equals, hashCode, toString과 같은 Object 메서드를 재정의할 수 없기 때문이다.

<br/>

골격 구현은 기본적으로 상속해서 사용하는 걸 가정하므로 아이템 19에서 이야기한 설계 및 문서화 지침을 모두 따라야 한다.

<br/>
<br/>

## 단순 구현

<center class="half">
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FT0xHN%2FbtrXa27I7kL%2FIKyafQxmiwcxOsnryFll80%2Fimg.png"
    width="100%"
    height="100%"
  />
</center>

단순 구현은 골격 구현의 작은 변종으로, AbstractMap.SimpleEntry가 좋은 예다. 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다. 쉽게 말해 동작하는 가장 단순한 구현이다. 이러한 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.