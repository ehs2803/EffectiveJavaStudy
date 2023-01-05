# 불변 클래스

- 인스턴스 내부 값을 수정할 수 없는 클래스
- 간직된 정보가 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않습니다.
- 가변 클래스보다 설계하고 구현하기 쉬우며, 오류가 생길 여지도 적습니다
- 상태를 변하지 않기 때문에 멀티 스레드 환경에서 안전합니다.

# 불변 클래스의 5가지 규칙

1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않습니다.
    1. 수정자(Setter)와 같이 필드 변수를 변경시키는 메서드를 작성해서는 안됩니다.
2. 클래스를 확장할 수 없도록 해야 합니다.
    1. 하위 클래스에서 의도치 않게 객체 상태를 변경할 수 있는 가능성을 막아야 합니다.
    2. ec) final class로 상속 통제, 정적 팩토리 메서드를 통해 생성자 통제
3. 모든 필드 final로 선언합니다.
    1. 명시적으로 불변을 선언하는 방법, 작성자의 의도가 불변 이라는 것을 명확하게 드러낼 수 있는 방법
4. 모든 필드를 private로 작성합니다.
    1. 클라이언트가 직접 필드에 접근해 수정하는 것을 막아줍니다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 합니다.
    1. 클래스에서 가변 객체를 참조하는 필드가 하나라도 있으면 클라이언트에서 그 객체의 참조를 얻지 못하게 해야 합니다.
    2. 내부의 가변 객체 필드를 반환하는 메서드가 있다면 방어적 복사를 실행 해야 합니다.

/방어적 복사

```java
public final class Period {
	private final Date start;
	private final Date end;

	public Period(Date start, Date nd) {
		if (start.compareTo(end) > 0)
			throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
		this.start = start;
		this.end = end;
	}

// Date start의 방어적 복사
	public Date start() {
		return new Date(start.getTime());
	}

// Date end의 방어적 복사
	public Date end() {
		return new Date(end.getTime());
	}
}
```

# 불변 클래스의 장점

```java
public final class Complex {

    private final double realNumber; // 실수부
    private final double imaginaryNumber; // 허수부

    public Complex(double realNumber, double imaginaryNumber) {
        this.realNumber = realNumber;
        this.imaginaryNumber = imaginaryNumber;
    }

    /**
     * 덧셈 연산
     */
    public Complex plus(Complex c) {
        return new Complex(realNumber + c.realNumber, imaginaryNumber + c.imaginaryNumber);
    }
}
```

불변 객체는 단순합니다. 생성된 시점부터 소멸되는 시점까지 상태가 동일하기 때문이죠. 

때문에 예측이 쉽고 **멀티스레드 환경에 안전하기 때문에 별도로 동기화 작업을 할 필요도 없습니다.** 또한 자유롭게 불변 객체를 공유할 수 있으며 불변 객체끼리는 내부 데이터를 공유할 수 있습니다.

```java
// 코드 일부
public class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum;
    final int[] mag;

    // ...코드 생략

    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }
}
```

예를 들어 `BigInteger` 클래스를 살펴보면 부호(sign)와 크기(magnitude)를 각각의 필드로 표현합니다. 크기는 같고 부호만 반대로 표현하는 `negate` 메서드를 보면 새로운 BigInteger를 생성하는데, 아래와 같이 가변인 배열을 복사하지 않고 원본 인스턴스와 공유하여 사용합니다.

그리고 불변 객체는 **그 자체만으로 실패 원자성을 제공합니다.** 그러니까 **예외가 발생한 이후에도 그 객체는 여전히 동일한 상태를 보장**합니다.

# 불변 클래스의 단점

불변 클래스를 통해 만들어지는 불변 객체의 경우, **값이 다르면 반드시 독립된 객체로 만들어야 합니다.**

만약 백만 비트짜리 BigInteger 객체에서 비트 하나를 바꿔야 한다고 가정해 보겠습니다. 이 경우 불변 클래스인 BigInteger 클래스는 비트 하나를 바꾼 새로운 객체를 생성하게 됩니다. 단지 하나의 비트만 바뀐 백만 비트 짜리 BigInteger 객체를 말이죠. 

이를 해결하기 위해서 가변 동반 클래스를 제공합니다. 예를 들어 불변인 `String` 클래스의 가변 동반 클래스로 `StringBuilder` 가 있습니다.

# 불변 클래스를 생성하는 방법

클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야 합니다. 이를 위해 본문에선 두 가지 방법을 제시합니다.

1. final class로 선언해 상속 제한하기
2. 생성자 대신에 정적 팩터리를 이용하기0

```java
public class Complex {
    // 클래스에 final이 없다.

    private final double realNumber; // 실수부
    private final double imaginaryNumber; // 허수부

    // 생성자가 private
    private Complex(double realNumber, double imaginaryNumber) {
        this.realNumber = realNumber;
        this.imaginaryNumber = imaginaryNumber;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

**생성자가 private**이므로 클라이언트에서 바라본 이 객체는 **사실상 final** 입니다. 다른 패키지에서는 이 클래스를 확장하는 것조차 불가능합니다. 이러한 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고 객체 캐싱과 같은 기능을 추가하여 성능을 끌어올릴 수도 있습니다.

# 정리

- 접근자 메서드(getter)가 있다고 무조건 수정자 메서드(setter)를 만들어야 하는 것은 아닙니다. 꼭 필요한 경우가 아니라면 **클래스는 불변(immutable)**이어야 합니다.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이는 것이 좋습니다. 객체가 가질 수 있는 상태의 개수가 줄어드는 것은 그 객체를 예측하기가 쉬워지고 오류가 발생할 가능성도 줄어들게 됩니다.