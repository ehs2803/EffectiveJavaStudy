# Item 19. 상속을 고려해 설계하고 문서화하라. 그렇지 않았다면 상속을 금지하라

> 클래스를 확장해야 할 명확한 이유가 떠오르지 않는다면 상속을 금지하는 편이 나을 것이다.  
> 상속을 금지하려면 클래스를 fianl로 선언하거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.

이번장에서는 상속을 허용하는 클래스가 지켜야 할 제약에 대해서 다룬다. 

---

아이템 18에서는 상속을 염두에 두지 않고 설계했고 상속할 때의 주의점도 문서화해놓지 않은 '외부' 클래스를 상속할 때의 위험을 경고했다. 여기서 '외부'란 프로그래머의 통제권 밖에 있어서 언제 어떻게 변경될지 모른다는 뜻이다. 그렇다면 상속을 고려한 설계와 문서화란 정확히 무엇일까

</br>

## 상속을 고려한 설계와 문서화

우선, 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다. 달리 말하면, **상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.** 클래스의 API로 공개된 메서드에서 클래스 자신의 또 다른 메서드를 호출할 수도 있다.(아이템 18 addAll) 그런데 재정의가 가능한 메서드라면 그 사실을 api 명세에 적시해야 한다.

> 재정의 가능 메서드란  
>         - public과 protected 메서드 중 final 이 아닌 모든 메서드

</br>

### **내부 메커니즘을 남겨라**

API 문서의 메서드 설명 끝에서 Implementation Requirements로 시작하는 절을 종종 볼 수 있는데, 그 메서드의 내부 동작 방식을 설명하는 곳이다. 이 절은 메서드 주석에 @implSpec태그를 붙여주면 자바독 도구가 생성해 준다. (자바 8에서 처음 도입되었다.)

</br>

- java.util.AbstractCollection의 remove 메서드

<center class="half">
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQ5Upi%2FbtrWx89ZTQa%2FCUgTyJSqK4KfNAklt8Ncn0%2Fimg.png"
    width="49%"
    height="50%"
  />
    <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlKe5K%2FbtrWxpqAsck%2FwGrFumKbVYpO1dPZXRA5U1%2Fimg.png"
    width="48%"
    height="50%"
  />
</center>

</br>


Implementation Requirements의 내용을 확인해 보자

> 이 구현은 지정된 요소를 찾는 컬렉션을 반복합니다. 요소를 찾으면 **iterator의 remove 메서드를 사용**하여 컬렉션에서 요소를 제거합니다. 이 컬렉션의 반복자 메서드에서 반환된 반복자가 remove 메서드를 구현하지 않고 이 컬렉션에 지정된 개체가 포함되어 있는 경우 이 구현은 UnsupportedOperationException을 throw 합니다.

</br>

이 설명에 따르면 remove 메서드는 iterator의 remove를 사용한다고 명시함으로써, iterator 메서드에 영향을 받는다는 사실을 알려준다. 이는 아이템 18에서 다룬 HashSet을 상속하여 add를 재정의한 것이 addAll에 까지 영향을 준다는 사실을 알 수 없었던 것과 대조적이다.

</br>
</br>

### **Protected 메서드 형태로 공개하라**

클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 proteced 메서드 형태로 공개해야 할 수도 있다. 드물게는 proteced 필드로 공개해야 할 수도 있다.

</br>

- java.util.AbstractList의 removeRange, clear 메서드
<center class="half">
  <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FK1NXp%2FbtrWCHiVDFX%2FTJcAzkZxzMvf6X9KzErWk0%2Fimg.png"
    width="45%"
    height="50%"
  />
    <img
    src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQxwvi%2FbtrWCVuuJcu%2FLekZtwsJ7VGeZgjHSM4VcK%2Fimg.png"
    width="50%"
    height="50%"
  />
</center>

</br>

List 구현체의 최종 사용자는 removeRange 메서드에 관심이 없다. 그럼에도 이 메서드를 제공한 이유는 단지 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서다. removeRange 메서드가 없다면 하위 클래스에서 clear 메서드를 호출하면 제곱에 비례해 성능이 느려지거나 부분리스트의 메커니즘을 밑바닥부터 새로 구현해야 했을 것이다.

</br>
</br>

## 상속을 허용하는 클래스가 지켜야 할 제약

### **반드시 하위 클래스를 만들어 검증하라**

상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다. 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 사실 private 이여야 할 가능성이 크다.

</br>
</br>

### **재정의 가능 메서드를 호출해서는 안 된다.**

상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행된다. 즉 하위 클래스에서 재정의 한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 그렇기 때문에 이때 그 재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않을 것이다.

</br>

**예제코드**

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() { ... }
}

public final class Sub extends Super {
    Sub() {
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```
</br>

이 프로그램이 instant를 두 번 출력하리라 기대했겠지만, 첫 번째는 null을 출력한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출하기 때문이다. 

</br>
</br>

## Clonable과 Serializable 인터페이스와 상속

clone과 readObject 메서드는 생성자와 비슷한 효과를 낸다. 따라서 상속용 클래스에서 Cloneable이나 Serializable을 구현할지 정해야 한다면, 이들을 구현할 때 따르는 제약도 생성자와 비슷하다는 점에 주의하자.

</br>

**즉, clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다. 이는 프로그램 오작동으로 이어질 수 있다**

-   readObject의 경우 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다.
-   clone의 경우 하위 클래스의 clone메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다. 특히 clone이 잘못되면 **원본객체에 피해를 줄 수 있다.**

</br>

**Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메서드를 갖는다면 이 메서드들은 private이 아닌 protected****로 선언해야 한다**. private으로 선언한다면 하위 클래스에서 무시되기 때문이다. 이 역시 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나다.

</br>
</br>

## 상속을 금지하는 방법

클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당함을 알았다. 절대 가볍게 생각하고 정할 문제가 아니다. 다음 장에서 볼 인터페이스의 골격 구현처럼 상속을 허용하는게 명백히 정당한 상황이 있고, 불변 클래스 처럼 명백히 잘못된 상황이 있다.  

</br>
  
**이러한 문제들을 해결하는 가장 좋은 방법은 상속용으로 설계하지 않은 클래스의 상속 자체를 금지하는 것이다(아이템 17)**

-   클래스를 final로 선언한다.
-   모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리를 만들어준다.

</br>
</br>

---

### **그래서 결론은...** 

#### 상속용으로 설계하지 않은 클래스는 상속을 금지해라

#### 하지만... 

구체 클래스가 표준 인터페이스를 구현하지 않았는데 상속을 금지하면 상당히 불편하다. 

#### 꼭 상속을 해야겠다면?

클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 즉 재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거하고 이사실을 문서로 남겨놓아라.