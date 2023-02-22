# item 26. 로 타입은 사용하지 말라

```java
public class Test {
    public static void main(String[] args) {
        List numbers = new ArrayList();
        numbers.add(10);
        numbers.add("text data");

        for(Object number : numbers) {
            System.out.println((Integer)number);
        }
    }
}
```

raw 타입 : 타입을 정의할 수 있음에도 정의하지 않은 경우

어떤 데이터 타입을 처리하는지 알 수 없다. Runtime시  문제가 발생

타입을 알 수 없는 경우는 casting을 해야 하지만 아래 코드처럼 타입을 아는 경우는 캐스팅이 필요하지 않다.

```java
public class Test {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList();
        numbers.add(10);
        numbers.add("text data");

        for(Object number : numbers) {
            System.out.println("number = " + number);
        }
    }
}
```

타입 매개변수 : 타입 등 타입이 정해진 한정적 타입

? : 비 한정적 타입 ,wild card type
