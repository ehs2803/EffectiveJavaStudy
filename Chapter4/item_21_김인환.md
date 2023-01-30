## 아이템 21.인터페이스는 구현하는 쪽을 생각해 설계하라

- 자바 8 전에는 인터페이스에 메서드를 추가시키면, 기존 구현체를 깨뜨릴 수 밖에 없음
- 이후 자바 8에 디폴트 메서드가 추가되면서 쉽게 메서드를 추가시킬 수 있게 되었지만, 여전히 위험성은 존재

- 디폴트 메서드를 추가한다는 것은 구현체에 대한 아무런 정보 없이 무작정 메서드가 삽입 되는 것
  - 대부분 잘 작동하지만, 모든 상황에서 잘 작동되는 디폴트 메서드를 작성하는 것은 어려움
  
### 디폴트 메서드에 의한 문제점 예시
- 자바 8 Collection 인터페이스의 removeIf 메서드

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


- 만약 위 메서드를 [아파치 커먼즈 라이브러리의 SynchronizedCollection](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/collection/SynchronizedCollection.html) 클래스와 함께 사용한다면 문제가 발생할 수 있음
  - SynchronizedCollection 클래스는 모든 메서드에서 클라이언트가 제공한 객체로 락을 걸어 동기화 기능을 추가로 제공하는 래퍼 클래스
  - 그러나, removeIf 메서드는 재정의되어 있지 않음. 즉 removeIf 메서드의 경우 클래스의 본래 의도와는 다르게 동기화를 제공하지 못함 (책이 쓰여진 시점에는)

  
- 직접 찾아봤더니 4.4 버전부터 removeIf 가 재정의된 것이 확인됨
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


### 결론
- 아무튼, 이번 예시 외에 언어 차원의 변화를 따라가지 못하고 여전히 수정되지 않고 있는 구현체들이 있음
- **디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있음**
  - 실제로 자바 8에서 추가된 많은 디폴트 메서드들이 기존에 짜여진 다른 자바 코드에 영향을 끼치고 있음


- 디폴트 메서드라는 도구가 유용하지만, 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피할 것
  - 대신에, 아이템 20에서 정리했듯이 새로운 인터페이스를 만드는 것으로 해결하면 더 좋을 듯 싶음
  
- 핵심은 **인터페이스를 설계할 때는 세심한 주의가 필요**
  - 릴리스 전에 반드시 테스트하기
  - 서로 다른 방식으로 최소한 세가지는 구현해보기
  - 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어보기