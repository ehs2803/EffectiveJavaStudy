# 비검사 경고를 제거하라

### 할 수 있는 한 모든 비검사 경고를 제거하라.

[비검사 경고가 발생하는 코드]

```java
Set<Lark> exaltation = new HashSet(); 
```
위 코드는 비검사 경고가 발생한다. 컴파일러의 안내에 따라 코드를 수정하면 비검사 경고가 사라진다.

하지만, 컴파일러의 안내와 달리 다이아몬드 연산자만으로도 비검사경고를 해결할 수 있다. 그러면 컴파일러가 올바른 실제 타입 매개변수(이 경우 Lark)를 추론해준다.


[비검사 경고가 발생해결]

```java
Set<Lark> exaltation = new HashSet<>();
```
이렇게 비검사 경고를 해결하면 타입 안정성이 보장된다.


### @SuppressWarnings("uncheck")를 이용한 비검사 경고 제거

만약, 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked")를 이용해 비검사 경고를 숨기자.

만약, 타입 안전하다고 검증된 코드의 검사를 그대로 두면 진짜 문제를 알리는 경고 코드를 구분하기 쉽지 않다.

또한 타입 안전함을 검증하지 않은 채 경고를 숨기면 잘못된 보안인식을 심어주는 꼴이된다.


### @SuppressWarnings("uncheck") 사용범위

@SuppressWarnings("unchecked")는 가능한 좁은 범위에 적용하자.

@SuppressWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.

한줄이 넘는 메서드나 생성자에 @SuppressWarnings가 달려있다면 지역변수나 아주 짧은 메서드 혹은 생성자로 옮기자.

절대로 클래스 전체에 적용해서는 안된다.

[기존 ArrayList의 toArray 메서드]

```java
public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
}
```
위 코드를 컴파일하면 @copyOf()@ 부분에서 경고가 발생한다. 이 경고를 제거하려면 지역변수를 추가해야 한다.

return문에는 @SuppressWarnings("unchecked")를 다는게 불가능하기 때문이다.

[지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.]

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // 생성한 배열과 매개변수로 받은 배열이 모두 T[]로 같으므로
        // 올바른 형변환이다.
        @SuppressWarnings("unchecked") 
        T[] result = (T[]) Arrays.copyOf(elementData, size, a.getClass()); 
        return result
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
이 코드는 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소로 좁혔다.

@SuppressWarnings("unchecked") 에너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야한다.

다른 사람이 해당 코드를 이해하는데 도움이되며, 그 코드를 잘못 수정해 타입 안정성을 잃는 상황을 줄여주기 때문이다.

### 정리

비검사 경고는 런타임에 @ClassCastException을 일으킬 수 있는 잠재적 가능성을 의미하니 가능한한 제거하자.

만약, 경고를 제거하기 힘들다면 타입 안정성을 증명하고 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨기고 그 근거를 주석으로 남기자.
