# 1. 메모리 누수에 주의하라

C,C++처럼 메모리를 직접 관리해야 하는 언어를 쓰다가 자바처럼 가비지 컬렉터를 갖춘 언어로 넘어오면 GC가 메모리를 관리해 주기 때문에 메모리에 신경을 쓰지 않아도 된다고 오해할 수 있습니다.

# 메모리 누수가 일어나는 위치는 어디인가?

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 코드를 보면 잘 돌아가는 Stack 클래스 처럼 보이지만 사실은 지속적으로 메모리 누수가 발생하고 있습니다.

위 코드에서는 Stack이 커졌다가 줄어들었을 때 Stack이 객체들의 다 쓴 참조를 가지고 있기 때문에 Stack에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않습니다.

![Stack 메모리 누수 PNG](https://user-images.githubusercontent.com/90227655/206167818-250940e0-0425-4d04-a6c1-1bfa5178da46.png)


결국 이 Stack이 객체들의 다 쓴 참조를 계속해서 가지고 있게 되고 살려둔 객체가 참조 하는 다른 모든 객체들도 메모리 회수하지 못하는 상태가 되어 GC의 활동과 메모리 사용량이 늘어나면서 점점 성능이 저하되게 됩니다.

# 2. null을 통한 객체 참조 해제

위와 같은 문제를 해결하는 것은 간단한데, **의도적으로 객체의 할당을 해제** 시켜주면 됩니다.

```java
public class Stack {

    // ...

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
}
```

![Stack 메모리 누수 해결 PNG](https://user-images.githubusercontent.com/90227655/206167797-1250c323-c89e-46b7-bfae-a771c8e071b9.png)


이렇게 null을 통해 참조를 해제한다면, 프로그램이 null처리한 참조를 실수로 사용하려 할 때NullPointerException을 던지며 종료되기 때문에 오류를 조기에 발견하는 장점도 가질 수 있게 됩니다.

하지만, 본문에서는 null을 통해 의도적으로 객체 참조를 해제하는 상황이 아주 예외 적인 상황이어야 한다고 설명합니다.

모든 객체를 다 쓰자마자 일일이 null 처리하는 것은 프로그램을 필요 이상으로 지저분하게 만들 뿐, 가장 좋은 참조 해제 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것입니다. 따라서 변수의 범위를 항상 최소가 되게 정의해야 합니다(아이템 57)

# 3. 메모리 누수를 주의해야 하는 상황

### 1. 자기 메모리를 직접 관리하는 클래스

위 스택의 경우와 같이 스스로 메모리를 관리하는 클래스의 경우 원소를 다 사용한 즉시 원소가 참조한 객체들은 null처리 해주어야 합니다.

### 2. 캐시 메모리

캐시 메모리에 객체를 넣고, 사용이 끝난 후에도 메모리 참조를 해제하지 않는 경우가 많다고 합니다.

- WeakHashMap을 사용해 캐시를 생성하자. 다 쓴 엔트리는 즉시 자동으로 제거된다.
- 시간이 지날수록 엔트리의 가치를 떨어트리고, 주기적으로 가치가 떨어진 엔트리를 정리해 주자.

### 3. 리스너(Listener) 혹은 콜백(Callback)

클라이언트가 콜백을 등록만 하고 해지하지 않는 경우 계속해서 쌓이게 됩니다. 이를 해결하기 위해 **콜백을 약한 참조로 저장**해 가비지 컬렉터가 수거해 갈 수 있게 합니다(WeakHashMap에 키로 저장).

# 정리

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
