# 아이템 20.추상클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘으로 인터페이스와 추상클래스가 존재한다.

자바8부터 인터페이스에 default method를 지원하여 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

둘의 가장 큰 차이는 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 반면, 인터페이스는 어떤 클래스를 상속했든 같은 타입으로 취급 받는다.

인터페이스 - 다중 상속이 가능하고 구현한 클래스와 같은 타입으로 취급음. Java8 부터 default 메서드 제공
추상클래스 - 다중 상속이 불가하고, 구현체와 상하관계에 있다.

# 인터페이스 장점


### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현할 수 있다

인터페이스 - 인터페이스의 추상 메서드를 추가하고, 클래스에 implements 구문을 추가하여 구현체임을 알린다.
추상클래스 - 계층 구조상 두 클래스의 공통 조상이어야 하며, 새로 추가된 추상 클래스의 모든 자손이 상속하게 된다.

### 믹스인 정의에 적합하다.

추상 클래스는 단일 상속만 가능하기 때문에 기존 클래스에 덧씌울 수 없다.

### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java
public interface Singer {
    AudioClip sing(Song s);
}

public interface SongWriter {
    Song compose(int charPosition);
}

public interface SingSongWriter extends Singer, SongWriter {
    AudioClip strum();
    void actSensitive();
}
```
```java
public abstract class Singer {
    abstract AudioClip sing(Song s);
}

public abstract class SongWriter {
    abstract Song compose(int charPosition);
}

public abstract class SingerSongWriter {
    abstract AudioClip sing(Song s);
    abstract Song compose(int charPosition);
    abstract AudioClip strum();
    abstract void actSensitive();
}
```
추상 클래스로 만들면 다중상속이 불가하여 새로운 추상클래스를 만들어서 클래스 계층을 표현할 수 밖에 없다.

따라서 이 계층구조를 만들기 위해서는 많은 조합이 필요하게 된다.


# 디폴트 메서드 제약

자바8부터 인터페이스에서도 메서드를 구현할 수 있게 되었다. default 메서드. 다만, 아래와 같은 규칙을 지켜줘야 한다.

- @implSpec 자바독 태그를 붙여 사용하려는 default 메서드를 문서화한다.
- equals와 hashCode는 default 메서드로 제공해서는 안된다.
- 인스턴스 필드를 가질 수 없다.
- public이 아닌 정적 멤버를 가질 수 없다.
- 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

# 추상 골격 클래스(Skeletal Implementation)

인터페이스로는 타입을 정의하고 디폴드 메서드도 제공한다. 골격 구현 클래스는 나머지 메서드들까지 구현한다.

이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 인터페이스를 구현하는데 필요한 일이 대부분 완료된다.(템플릿 메서드 패턴)

# 시뮬레이트한 다중 상속(Simulated Multiple Inheritance)

골격 구현 클래스를 우회적으로 이용하는 방식이다.
인터페이스를 구현한 클래스에서 골격구현을 확장한 private 내부 클래스를 정의하고 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

아이템 18에서 다룬 내용과 비슷한 방식이다.

```java
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;

  public ForwardingSet(Set<E> s) {
    this.s = s;
  }

  public void clear() {
    s.clear();
  }
  ... 중략 ...
}
```

### 골격 구현 작성방법

먼저 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정한다.

이 기반 메서드들을 사용해 직접 구현할 수 있는 메서드들을 모두 디폴트 메서드로 제공한다.

기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들은 작성해 넣는다.

다음과 같이 구현한다.

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    @Override public boolean equals(Object o) {
        if (o == this) {
            return true;
        }

        if (!(o instanceof Map.Entry)) {
            return false;
        }

        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey()) && Objects.equals(e.getValue(), getValue());
    }

    @Override public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

# 결론

일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.

복잡한 인터페이스라면 구현하는 수고드를 덜어주는 골격 구현을 고려해보자.
