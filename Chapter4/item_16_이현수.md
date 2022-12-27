# 아이템 16.public 클래스에서는 public 필드가 아닌 다른 접근자 메서드를 사용하라

```java
public class Point{
  public double x;
  public double y;
}
```
클라이언트에서 필드에 직접 접근할 수 있으므로 캡슐화의 이점을 제공하지 못한다는 단점

- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.


public 필드로만 구성되어 있기 때문에 내부 표현을 변경하기 위해서는 API의 필드를 변경해야 한다. (메소드가 존재할 땐 파라미터에 따라 내부 표현이 변경 가능)

- 불변식을 보장할 수 없다.


클라이언트에서 직접적으로 필드에 접근하고 있으므로 클라이언트에 의해 언제든지 변경이 가능하다.

- 외부에서 필드에 접근할 때 부수적인 로직을 추가할 수 없다.

Point.x 라는 필드를 조회했을 때 부수적인 로직(Ex. 연산 로직)을 추가할 수가 없다.

```java
class Point {
  private double x;
  private double y;

  public Point (double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```
클래스의 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 제공하게 된다.

getter/setter 혹은 또 다른 메소드를 통해 로직을 언제든 추가할 수 있다.

이 전 소스 처럼 public 클래스가 필드값을 공개(pubilc 접근자)하게 되면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현방식을 마음대로 바꿀 수 없게 된다.

---

하지만, package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다고 해도 문제될 것이 없다.

그 이유는 같은 패키지 안에서 어떤 특정 이유 때문에 사용하던가 탑 레벨 클래스에서만 접근하기 때문에 위에서 언급한 단점들이 나타날 이유가 없어 문제될 것이 없다.

# 결론 

public 클래스의 경우 가변 필드의 접근 제한자를 public으로 두면 안 되며, 불변 필드라고 해도 덜 위험하지만 안심할 수 없다. 만약 가변 필드를 노출하고 싶을 땐, package-private 클래스나 private 중첩 클래스를 활용해라.
