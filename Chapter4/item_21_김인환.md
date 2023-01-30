## 인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 전에는 인터페이스에 메서드를 추가시키면, 기존 구현체를 깨뜨릴 수 밖에 없었다.
- 이후 자바 8에 디폴트 메서드가 추가되면서 쉽게 메서드를 추가시킬 수 있게 되었지만, 여전히 위험성은 존재한다.

- 디폴트 메서드 추가는 구현체에 대한 아무런 정보 없이 무작정 메서드가 삽입 되는 것이다.
  - 대부분 잘 작동하지만, 모든 상황에서 잘 작동되는 디폴트 메서드를 작성하는 것은 어렵다.
  
#### 자바 8의 Collection 인터페이스의 removeIf 예시

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


위 메서드는 아파치 커먼즈 라이브러리의 SynchronizedCollection 클래스에서 문제를 일으킨다. (지금은 해당 문제가 해결된 듯하다.)
- SynchronizedCollection 클래스는 기존 java.util 의 Collections.synchronizedCollection 클라이언트가 제공한 객체로 락을 거는 기능을 추가로 제공한다.

- https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/collection/SynchronizedCollection.html

```java
/**
* @since 4.4
*/
@Override
public boolean removeIf(final Predicate<? super E> filter) {
    synchronized (lock) {
        return decorated().removeIf(filter);
    }
}

```