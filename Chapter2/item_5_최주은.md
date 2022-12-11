# 아이템5_자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존합니다. 가령 맞춤법 검사기는 사전에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 드물지 않게 볼 수 있습니다.

</br>

**아이템 4. static 유틸 클래스 사용** 

```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

	//static 메서드에서 이 자원을 사용함으로 static선언
    private static final Lexicon dictionary = new KoreanDicationry();
	
    //private 생성자 - 객체 생성 방지
    private SpellChecker() {
        // Noninstantiable
    }

    // 모든 유틸 메서드는 public 정적 메서드
    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }


    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

</br>


**아이템 3. 싱글톤 사용** 

```java
//싱글턴을 잘못 사용한 예 - 유연하지않고 테스트 하기 어렵다.
public class SpellChecker {
    private Lexicon dictionary = dictionary = new KoreanDicationry();
    
    //private 생성자 - 객체 생성 방지
    private SpellChecker() {
        // Noninstantiable
    }
    
    //static 인스턴스 -> 싱글톤
    public static SpellChecker INSTANCE = new SpellChecker() {
    };
    
    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }
    
    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
    
    public static void main(String[] args){
        SpellChecker.INSTANCE.isValid("hello world")
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

</br>


위 두 개의 코드 모두 사전을 하나만 사용한다고 가정을 하고 있기 때문에 좋은 코드는 아닙니다.

실전에서는 사전이 언어별로 따로 있고 특수 어휘용 사전들도 별도로 있기도 하기 때문입니다.

그렇기 때문에 사전 하나로  모든 쓰임을 대응할 수 있기를 바라는 것은 너무 단순한 생각이라고입니다.

그렇다면 위에 SpellChecker(맞춤법 검사기)가 여러 사전을 사용할 수 있도록 변경해 보겠습니다.

간단하게 dictionary 필드에서 fianl 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수 있습니다.

하지만 이 방식은 어색하기도 하고 오류를 내기 쉬우며 멀티 스레드 환경에서는 사용할 수 없습니다.

</br>


 **fianl 한정자를 제거하면 멀티 스레드 환경에서 사용하지 못한다?**

> 멀티 스레드 환경에서는 여러 스레드(Thread)가 동시에 하나의 자원에 접근할 수 있습니다. (스레드는 레지스터와 stack만 독립적으로 갖고 나머지는 공유합니다) 여러 스레드가 동시에 공유 자원(method area, heep area)에 대한 수정을 할 경우 동시성 문제가 발생할 수 있습니다.  
>   
> 예제 1.  
> dictionary는 static객체이기 때문에 method area에 저장되고 여러 스레드에서 dictionary에 접근하여 사용할 때 전부 같은 값을 공유합니다. 메서드를 통해 다른 사전으로 변경한다 하더라도 다른 스레드에 의해 유지되기 힘들기 때문에  thread safe하지 않습니다.  
>   
> 예제 2.  
> dictionary는 static객체는 아니지만 싱글턴으로 구현이 되어있습니다. 즉 인스턴스를 단 하나만 생성할 수 있습니다. 인스턴스는 heep area 저장되어 위와 마찬가지로 여러 스레드에서 에서 dictionary에 접근할 때 하나의 값을 공유하게 되므로 thread safe하지 않습니다

</br>
</br>


그렇기 때문에 **사용하는 자원에 따라 동작이 달라지는 클래스에서는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않습니다**. 대신 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(dictionary)을 사용할 수 있어야 합니다.

이조건을 만족하는 간단한 방식이 있는데 바로 인스턴스를 생성할 때 생성자의 필요한 방식을 넘겨주는 방식입니다. 이는 의존성 객체 주입의 한 형태로, 맞춤법 검사기(SpellChecker)를 생성할 때 의존성 객체인 사전(dictionary)을 주입해주면 됩니다.

의존성 객체 주입은 유연성과 테스트 용의성을 높여 줍니다 

</br>

**생성자를 통한 의존성 주입을 사용한 SpellChecker**

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
    	this.dictionary = Objects.requireNonNull(dictionary);
    }
        
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
    
    public static void main(String[] args){
    	Lexicon lexicon = new KoreanDicationry()
        SpellChecker spellchecker = new SpellChecker(lexicon);
        spellchecker.isValid("hello world")
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

위와 같은 의존성 주입은 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동합니다 또한 불변(final 한정자)을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 합니다 

의존성 주입은 생성자, 스태틱 팩토리(아이템 1) 그리고 빌더(아이템 2) 모두에도 똑같이 적용할 수 있습니다.

이 패턴의 변형으로는 생성자에 자원 팩토리를 넘겨주는 팩토리 메서드 패턴 방식이 있습니다. 

팩토리란 호출될 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말합니다. Supplier <T> 인터페이스는 팩토리를 표현한 완벽한 예입니다.   
  
아래 방식은 클라이언트에서 제공한 팩토리가 생성한 타일들로 구성된 모자이크를 만드는 메서드입니다. 



```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

</br>


Supplier <T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(아이템 31)을 사용해 팩토리의 타입 매개변수를 제한해야 합니다. 위의 Supplier는 Tile의 상속을 받는 객체만 타입 변수로 받을 수 있습니다. 즉 Tile의 속성을 띄는 다양한 종류의 Tile을 외부에서 의존성 주입을 할 수 있다는 것입니다.

</br>

**Supplier를 입력받는 예**

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Supplier<Lexicon> dictionary) {
    	this.dictionary = Objects.requireNonNull(dictionary);
    }
        
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
    
    public static void main(String[] args){
    	Lexicon lexicon = new KoreanDicationry()
        SpellChecker spellchecker = new SpellChecker(() -> lexicon);
        spellchecker.isValid("hello world")
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

</br>


의존 객체의 주입이 유연성과 테스트 용의성을 개선해주긴 하지만, 의존성이 수 천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 합니다. 이런 어지러움은 대거(Dagger), 스프링(Spring) 같은 프레임워크를 사용하면 해소할 수 있습니다.

이들 프레임워크는 의존 객체를 직접 주입하도록 설계된 API를 알맞게 응용하여 사용하고 있습니다.

</br>


#### 정리

클래스가 내부적으로 하나이상의 자원의 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 게 좋습니다. 또한 이 자원들은 클래스가 직접 생성하게 해서도 안됩니다. 

대신 필요한 자원(혹은 그 자원을 만들어주는 팩토리를)을 생성자(혹은 정적 팩토리 빌더)에 넘겨줘야 합니다. 

흔히  의존성 객체 주입이라고 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용의성을 큰 폭으로 개선해줍니다.