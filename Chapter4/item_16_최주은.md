
# item 16. public 클레스에서 public 필드가 아닌 접근자 메서드를 사용하라

> public 클래스의 절대 가변 필드를 직접 노출하지 말아야 한다. 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가하자. 

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어긴 사례가 종종 있는데 대표적인 예가 java.awt.package의 Point와 Dimension 클래스이다.

내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다.

---

</br>

## public 클래스의 절대 가변 필드를 직접 노출하지 말아야 한다

아래와 같이 인스턴스 필드들을 모아놓는 일 이외에는 아무 목적도 없는 클래스를 작성할 때 필드는 public 이는 안된다.

```java
class Point {
    public double x;
    public double y;
}
```

</br>

이런 클래스는 데이터 필드에 직접 접근하여 수정이 가능하다. 따라서 캡슐화의 이점을 제공하지 못한다.

-   API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
-   불변식을 보장할 수 없다.
-   외부에서 필드에 접근할 때 부수 작업을 수행 할 수도 없다.

</br>

## 필드를 모두 private으로 바꾸고 public 접근자(getter)를 추가하자. 

위의 Point 클래스의 필드를 private으로 바꾸고 접근자를 추가했다.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```
</br>


-   public 클래스에서라면 이 방식이 확실히 맞는 방법이다.
-   패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

</br>


## package-private 클래스, private 중첩 클래스

-   package-private 클래스 혹은 private 중첩 클래스는 필드를 노출해도 상관없다.
-   그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.
-   이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.

</br>


## 불변 필드를 노출한 public 클래스

가변 필드가 아닌 불변 필드를 노출시키면 어떨까? 아래는 불변 필드를 노출한 public 클래스의 예이다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException(" 시간 : " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException(" 분 : " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```
</br>

직접 노출할 때의 단점이 조금 줄어들긴 하지만, 이것 역시 결코 좋은 생각이 아니다.

-   API를 변경하지 않고는 표현 방식을 바꿀 수 없다.
-   필드를 읽을 때 부수 작업을 수행할 수 없다.
-   단 불변식은 보장할 수 있다.

</br>


## 핵심 정리

-   public 클래스는 절대 가변 필드를 직접 노출해서는 안된다.
-   final로 선언된 불변 필드라면 노출해도 위험이 덜하지만 완전히 안심할 수는 없다.
-   pakcage-private 클래스 혹은 private 중첩 클래스라면 필드를 노출하는 편히 나을 때도 있다.