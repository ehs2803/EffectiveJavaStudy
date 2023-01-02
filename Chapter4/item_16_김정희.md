# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
-   public 클래스는 필드를 직접 노출하지 말 것.
    -   객체지향 프로그래밍에서는 필드를 private로 선언하고, getter를 추가한다.

```
public class Main {
    public double x;
    public double y;

    public static void main(String[] args) {
        Main main = new Main();
        main.x = 10;
        main.y = 10;
        System.out.println(main.x);
        System.out.println(main.y);
    }
}
```

위와 같은 코드는 캡슐화의 장점을 제공하지 못한다.

필드에 접근하는 방식을 메서드로 지정하면 필드명이 변경되어도 메서드 명을 바꿀 필요는 없어진다. 또한 메서드 안에서 부가적인 작업도 가능해진다.  
단, package-private, private 중첩 클래스라면 데이터 필드를 노출해도 무방하다.

```
public class Main {
    private double x;
    private double y;

    public Main(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        //부가작업
        return x;
    }
    public double getY() {return y;}

    public void setX(double x) {
        //부가작업
        this.x = x;
    }
    public void setY(double y) {this.y = y;}
}
```

만약 public으로 선언한 필드들(가변적)을 안전하게 쓰기 위해 필드가 사용되는 범위 안에서 값을 복사하여 사용해야 한다.

하지만 인스턴스를 만드는 방법은 불필요한 인스턴스를 만드는 것이므로 성능저하 요인이 될 수 있다.

```
public class Main {
    public double x;
    public double y;

    public static void main(String[] args) {
        Main main = new Main();
        main.x = 10;
        main.y = 10;

        doSomething(main);

        System.out.println(main.x);
        System.out.println(main.y);
    }

    private static void doSomething(Main main) {
        Main localMain = new Main();
        localMain.x = main.x;
        localMain.y = main.y;
    }
}
```

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 않아야 하는 규칙을 어긴 사례가 존재한다.

java.awt.package의 Dimension, Point클래스이다.
![image](https://user-images.githubusercontent.com/45592236/209675306-0b0d3ffc-00db-4301-91c6-5d892a17d464.png)
