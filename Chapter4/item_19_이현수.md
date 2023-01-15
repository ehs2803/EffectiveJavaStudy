# Item19- 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.

## 1. 상속을 고려한 문서화와 설계


메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다. 

즉, 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야한다.

재정의할 수 있는 메서드란 public과 protected 메서드 중 final이 아닌 모든 메서드.

![image](https://user-images.githubusercontent.com/65898555/212077868-253e4edb-6763-4239-bf0b-5785c5f211ce.png)

API문서의 메서드 설명 끝에서 종종 "Implementation Requiredments"로 시작하는 절을 볼 수 있다. 그 메서드 내부 동작 방식을 설명하는 곳이다. 이 절을 메서드 주석에 @implSpec 태그를 붙여주면 자바독 도구가 생성해준다.


## 2. hook 선별

내부 메커니즘을 문서로 남기는 것만이 상속을 위한 설계의 전부는 아니다. 효율적인 하위 클래스를 큰 어려움 없이 만들기 위해서 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메서드 형태로 공개해야할 수도 있다.

![image](https://user-images.githubusercontent.com/65898555/212078222-eb695737-330a-4f34-995c-eda89f731d6d.png)

위와 같이 AbstractList clear 메서드는 removeRange를 내부에서 호출하고 있다. 이유는 하위 클래스에서 리스트 구현 내부 구조의 이점을 잘 활용하여 removeRange 메서드를 활용한다면 clear 메서드의 성능을 향상 시킬 수 있기 때문이다.

 

이때 유의할 점은 removeRange의 접근 제한자가 protected로 설정되어있다는 것이다. 하위 클래스를 만들 때 전혀 쓰이지 않는 메서드는 private로 만들면 된다. 하위 클래스에서 사용할 일이 있다면 protected로 만들어준다. 상속용으로 설계한 클래슨느 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다.


## 3. 상속용 클래스의 생성자는 재정의 가능한 메서드를 호출해서는 안된다.

Super클래스에서 하위 클래스에서 재정의가 가능한 overrideMe 메서드를 호출한다.


'''java
public class Super {

    // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다
    public Super() {
        overrideMe();
    }

    public void overrideMe() { }
}
```
상속용 클래스 Super - 생성자가 재정의 가능 메서드를 호출함

```java
public class Sub extends Super {

    // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    // 재정의 가능 메서드. 상위 클래스의 생성자가 호출된다.
    @Override
    public void overrideMe() {
        System.out.println(instant);
    }
}
```
하위 클래스 Sub - 생성자에서 초기화하는 필드를 지니고 있고, 재정의한 메서드에서 이를 의존함

```java
public class Item19 {

    public static void main(String[] args) {

        Sub sub = new Sub(); // Super 생성자 호출 -> overrideMe() 호출 -> null 출력 -> Sub 생성자 호출
        sub.overrideMe(); // 현재 시간 출력

        /**
         * null
         * 2022-05-05T23:45:22.066694Z
         */
    }
}
```
- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행됨
- 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출됨
- 이때, 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값을 의존하므로 NullPointerException이 발생하게 됨
 


private, final, static 메서드는 재정의가 불가능하므로 생성자에서 안심하고 호출해도 된다.

## 4. 상속용으로 설계하지 않은 클래스는 상속을 금지해라

#### 보통의구체 클래스를 그 내부만 수정했음에도 이를 확장한 자식 클래스에서 문제가 생겼다는 버그 리포트를 받는 일이 드물지 않다. 상속을 금지하는 방법은 다음과 같다.

- 클래스를 final로 선언
- 모든 생성자를 private or package private로 지정하고, public 정적 팩터리를 만들어준다.

#### 대신 핵심 기능을 정의한 인터페이스를 구현해라

#### 표준 인터페이스를 구현하지 않았는데 상속이 필요하다면?
- 재정의 가능 메서드를 호출하는 사용 코드를 모두 제거하라 → 메서드를 재정의해도 다른 메서드의 동작에 영향을 주지 않음
- 먼저 각각의 재정의 가능 메서드의 내부 코드를 private 도우미 메서드로 옮기고, 이 도우미 메서드를 호출하도록 수정한다
- 그런 다음 재정의 가능 메서드를 호출하는 다른 코드들도 모두 이 도우미 메서드를 직접 호출하도록 수정한다

```java
public class Concrete {

    public Concrete() {}

    public void overrideMe() {
        System.out.println("overrideMe");
    }

    public void doSomething() {
        overrideMe();
    }
}
```
Concrete 클래스 - 재정의 가능 메서드가 그대로 노출됨
```java
public class BetterConcrete {

    public BetterConcrete() {}

    public void overrideMe() {
        helper();
    }

    private void helper() {
        System.out.println("overrideMe");
    }

    public void doSomething() {
        helper();
    }
}
```
BetterConcrete - 재정의 가능 메서드의 내용을 private 도우미 메서드로 옮김


# 결론 

상속용 클래스를 설계하기란 어렵다

따라서 상속이 분명하게 필요한 것이 아니라면 금지하라
