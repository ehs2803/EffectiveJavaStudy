# Comparable 인터페이스란?

- Comparable 인터페이스는 객체를 정렬하는데 사용되는 메서드인 compareTo를 정의하고 있다.

# Comparable 인터페이스 특징
![image](https://user-images.githubusercontent.com/65898555/207048668-2cb03338-3fb8-494b-b4da-fe8ab85c5eea.png)

- 자바에서 같은 타입의 인스턴스를 비교해야만 하는 클래스들은 모두 Comparable 인터페이스를 구현하고 있다.
- Boolean 타입을 제외한 래퍼 클래스와 알파벳, 연대같이 순서가 명확한 클래스들은 모두 정렬이 가능하다.
- 기본 정렬 순서는 작은 값에서 큰 값으로 정렬되는 오름차순이다.

# Comparable 인터페이스 구현

Comparable 을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 의미한다.

```java
public class Car implements Comparable<Car> {
    private static final String SPACE = " ";
    private static final String MODEL_YEAR = " 식 ";

    private String modelName;
    private int modelYear;
    private String color;

    public Car(final String modelName, final int modelYear, final String color) {
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.color = color;
    }

    @Override
    public String toString() {
        return this.modelYear + MODEL_YEAR + this.modelName + SPACE + this.color;
    }

    @Override
    public int compareTo(Car car) {
        return Integer.compare(this.modelYear, car.modelYear);
    }
}

public class Main {
    public static void main(String[] args) {
        Car car1 = new Car("그렌저", 2016, "검정색");
        Car car2 = new Car("쏘나타", 2020, "흰색");

        System.out.println(car1.compareTo(car2));
    }
}
```
```
-1
```

# Comparable 인터페이스의 compareTo 메서드

compareTo 는 해당 객체와 전달된 객체의 순서를 비교한다.

compareTo는 Object의 euqals와 두가지 차이점이 있다. compareTo는 equals와 달리 단순 동치성에 더해 순서까지 비교할 수 있으며, 제네릭하다.

Comparable을 구현했다는 것은 그 클래스의 인스턴스에 자연적인 순서가 있음을 뜻한다. 예를 들어, Comparable을 구현한 객체들의 배열은 Arrays.sort(a)로 쉽게 정렬이 가능하다.

# compareTo 메서드 일반규약

이 객체와 주어진 객체의 순서를 비교한다. 
이 객체가 주어진 객체보다 작으면 음의정수를 같으면 0을, 크면 양의 정수를 반환한다. 비교할 수 없는 타입이 주어지면 ClassCastException을 던진다.

- 아래 설명에서 sgn은 부호함수를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
- 만약, 기존 클래스를 확장한 구체 클래스가 값 필드를 추가했다면 compareTo규약을 준수할 방법이 없다. 따라서 이런 경우 조합(Composition)을 이용해 우회적으로 규약을 준수하자.

☑️ 대칭성

두 객체참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다.
Comparable을 구현한 클래스는 모든 x, y에 대하여 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.


☑️ 추이성

첫 번째가 두번째보다 크고 두 번째가 세번째보다 크면 첫번째는 세번째보다 커야한다.
Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y)>0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0 이다.

☑️ 반사성

크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.
Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.

☑️ equals

compareTo 메서드로 수행한 동치성테스트 결과가 equals와 같아야한다.
(x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. (주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.)


# equals와 compareTo의 차이
[compareTo와 equals가 일관되지 않는 BigDecimal 클래스]
```java
final BigDecimal bigDecimal1 = new BigDecimal("1.0");
final BigDecimal bigDecimal2 = new BigDecimal("1.00");

final HashSet<BigDecimal> hashSet = new HashSet<>();
hashSet.add(bigDecimal1);
hashSet.add(bigDecimal2);

System.out.println(hashSet.size());

final TreeSet<BigDecimal> treeSet = new TreeSet<>();
treeSet.add(bigDecimal1);
treeSet.add(bigDecimal2);

System.out.println(treeSet.size());
```
```
hashSet: 2
treeSet: 1
```
- HashSet과 TreeSet은 서로 다른 메서드로 객체의 동치성을 비교한다.
- HashSet은 equals를 기반으로 비교하기 때문에 추가된 두 BigDeciaml이 다른값으로 인식되어 크기가 2가 된다.
- 반면에, TreeSet은 compareTo를 기반으로 객체에 대한 동치성을 비교하기 때문에 같은값으로 인식되어 compareTo가 0을 반환하기 때문에 크기가 1이 된다.


# CompareTo 메서드 작성 요령

- Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo의 인수타입은 컴파일 시에 정해지기 때문에 입력 인수 확인이나 형변환을 할 필요가 없다.
- null을 인수로 넣으면 NullPointerException을 던져야한다.
- compareTo는 동치가 아닌 순서를 비교한다.
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator을 대신 사용한다.

# compareTo 메서드에서 관계연산자 (<, >)를 사용하지 않는것을 추천한다.
- 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 compare를 대신 이용한다.
- 관계연산자(<,>)는 오류를 유발할 가능성이 있기때문에 추천하지 않는다.


# 클래스에 핵심 필드 여러개일때 비교

클래스에 핵심필드가 여러개라면 가장 핵심적인 필드부터 비교하자.

비교 결과가 0이 아니라면, 즉 순서가 결정되면 바로 결과를 반환하면 된다. 똑같지 않은 필드를 찾을 때 까지 비교해나가도록 구현하면 된다.

[기본 타입 필드가 여럿일 때 비교자]
```java
public int compare(final PhoneNumber phoneNumber) {
        int result = Short.compare(areaCode, phoneNumber.areaCode);

        if (result == 0) {
            result = Short.compare(prefix, phoneNumber.prefix);
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber.lineNum);
            }
        }

        return result;
}
```

# Comparator 인터페이스

자바 8에서 Comparator 인터페이스가 일련의 비교자 생성 메서드와 메서드 연쇄방식으로 비교자를 생성할 수 있게 되었다.

- 비교자들은 compareTo 메서드를 구현하는데 활용될 수 있다. 이 방식은 간결하지만 약간의 성능 저하가 뒤따른다. 자바의 정적 임포트 기능을 활용하면 정적 비교자 생성 메서드들을 그 이름만으로 사용할 수 있어 코드가 훨씬 깔끔해진다.
- Comparator는 comparingInt와 thenComparingInt등의 숫자용 기본 타입을 커버하는 보조 생성 메서드들을 가지고 있다.
- comparing와 thenComparing이란 객체 참조용 비교자 생성 메서드 또한 가지고 있다.

[정적 생성 메서드들을 이름으로 활용한 방식을 적용한 compareTo 메서드]
```java
private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
                    .thenComparingInt(phoneNumber -> phoneNumber.prefix)
                    .thenComparingInt(phoneNumber -> phoneNumber.lineNum);

public int compareTo(PhoneNumber phoneNumber) {
	    return COMPARATOR.compare(this, phoneNumber);
}
```
- 위 코드는 비교자 생성 메서드 2개를 이용해 비교자를 생성한다. 첫 번째는 comparingInt 두 번째는 thenCompaingInt이다.
- comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메소드이다.
- thenComparingInt는 Comparator의 인스턴스 메서드로, int 키 추출 함수를 입력받아 다시 비교자를 반환한다. ( 이 비교자는 첫 번째 비교자를 적용한 다음 새로 추출한 키로 추가 비교를 수행한다.) thenComparingInt는 연달아 호출이 가능하다. 또한 호출할 때 타입을 명시하지 않는다. (자바의 타입 추론 능력으로 추론이 가능)

[추이성을 위반 - 해시코드 값의 차를 기준으로 하는 비교자]
```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
};
```
이 방식은 정수 오버플로를 일으키거나 부동소수점 계산방식에 따른 오류를 발생시킬 수 있어 사용하면 안된다. 따라서 아래의 두 방식을 대신 사용하도록 하자.

[정적 compare 메서드를 활용한 비교자]
```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
};
```
[정적 compare 메서드를 활용한 비교자]
```java
static Comparator<Object> hashCodeOrder = 
Comparator.comparingInt(o -> o.hashCode());
```

# 정리

순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.
compareTo 메서드에서 필드의 값을 비교할 때 <와> 연산자는 지양해야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator가 제공하는 비교자 생성 메서드를 이용하자.
