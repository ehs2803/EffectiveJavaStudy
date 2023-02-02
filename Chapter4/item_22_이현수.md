# 인터페이스는 타입을 정의하는 용도로만 사용하라

### 인터페이스 사용용도

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.

달리 말하면 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기하주는 것이다.

인터페이스는 반드시 위 용도로만 사용이 되어야한다.

### 상수 인터페이스

상수 인터페이스란 메서드없이 상수를 뜻하는 static final 필드로만 구성된 인터페이스를 말한다. 

그리고 이 상수들을 사용하려는 클래스에서는 정규화된 이름을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 한다.


```java
public interface CarConstants {
    public static final String INPUT_CAR_NAME_MESSAGE = "경주할 자동차 이름을 입력하세요.(이름은 쉼표(,) 기준으로 구분)";
    public static final String INPUT_ROUND_MESSAGE = "시도할 회수는 몇회인가요?";
}
```
위 코드는 상수 인터페이스 안티패턴으로 인터페이스를 잘못 사용한 예다.

클래스 내부에서 사용하는 상수는 내부구현에 해당한다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위이다.

클래스가 어떤 상수 인터페이스를 사용하던 사용자에게는 아무런 의미가 없고 오히려 혼란을 주게된다. 동시에 클라이언트 코드가 이 상수들에 종속되어 다음 릴리스에서 상수를 사용하지 않게 되더라도 여전히 상수 인터페이스를 구현해야 한다.

### 상수 공개 방법

대표적인 상수공개 방법 몇가지를 소개한다.

- 클래스나 인터페이스 자체에 추가하는 방법
- 열거 타입으로 공개하는 방법
- 유틸리티 클래스에 담아 공개하는 방법

### 1. 클래스나 인터페이스 자체에 상수 추가 방법
```java
public final class Integer extends Number implements Comparable<Integer> {
	...
	@Native public static final int   MIN_VALUE = 0x80000000;

	@Native public static final int   MAX_VALUE = 0x7fffffff;
    ...
}
```
특정 클래스나 인터페이스와 강하게 연관된 상수라면 클래스나 인터페이스 자체에 추가해야한다. 모든 기본 타입의 박싱 클래스가 대표적으로, Integer와 Double에 선언된 MIN_VALUE, MAX_VALUE 가 있다.

### 2. 열거 타입으로 공개하는 방법

열거타입으로 나타낼 수 있으면 Enum을 사용하면 된다.

### 3. 유틸리티 클래스에 담아 공개하는 방법

```java
public class CarConstants {
    public static final String INPUT_CAR_NAME_MESSAGE = "경주할 자동차 이름을 입력하세요.(이름은 쉼표(,) 기준으로 구분)";
    public static final String INPUT_ROUND_MESSAGE = "시도할 회수는 몇회인가요?";

    private CarConstants() {
        // 인스턴스화 방지
    }
}
```
상수 유틸리티 클래스

```java
import static item22.CarConstants.*;

public class Main {
    String printInputRoundMessage(){
        return INPUT_ROUND_MESSAGE;
    }
    
    String displayInputCarNameMessage(){
        return INPUT_CAR_NAME_MESSAGE;
    }
 ...   
}
```
만약 유틸리티 클래스 사용이 빈번하다면 정적 임포트로 클래스이름 생략이 가능하다.


### 정리

인터페이스는 타입을 정의하는 용도로만 사용하자. 인터페이스를 상수 공개용 수단인 상수 인터페이스로 사용하면 여러가지 단점이 있으므로 사용을 지양하자.

상수 인터페이스 단점

- 구현 방식이 외부에 제공하는 API에 드러나게 된다.
- 상수를 사용할 필요가 없어져도 바이너리 호환성때문에 계속 상수 인터페이스를 구현해야 한다.
- 사용되지 않을 수 있는 상수 때문에 클래스 네임 스페이스가 오염된다.

