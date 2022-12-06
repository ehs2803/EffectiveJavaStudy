# item 6. 불필요한 객체 생성을 피하라

### 예시1) 문자열

`String str = new String(”테스트”);`로 객체를 생성해서 사용하는 것이 아닌 문자열 리터럴 방식으로 사용해야한다.

리터럴로 String을 생성하면 String constant pool에 존재하게되고, new 연산자로 생성하게 되면 heap 영역에 데이터가 생성된다.

리터럴 방식으로 생성하게 되면 String 내부의 intern() 메서드가 호출되고 JVM은 내부적으로 hashmap 같은 String constant pool에 문자열을 넣고 캐싱한다.

동일한 문자열을 사용하면 상수 pool 에서 동일한 문자열을 참조하는 방식으로 사용

new로 하면 매번 새로운 문자열 객체를 만드므로 불필요한 객체가 생성된다. 그러므로 `Stirng str = “테스트”` 리터럴 방식을 사용해 문자열을 재사용하는것이 좋다.

객체가 같은 인스턴스인지 비교

```java
String hello = "hello";
String helloObj = new String("hello");
System.out.println(hello == helloObj);//false
System.out.println(hello.equals(helloObj));//true
```
→ 인스턴스가 동일하기때문에 불필요하게 만들 필요가 없다.

### 예시2) 정규식 Pattern

- 생성 비용이 비싼 객체라서 반복해서 생성하기 보다, 캐싱 하여 **객체를 재사용 하는 것이 좋다.**
- 정규식의 경우 객체 생성 시 생성 비용이 비싸다. = cpu 리소스 사용

문자열을 파라미터로 받아 정규 표현식을 통해 문자열을 검사할때 내부 메소드를 들여다 보면 검사 할때마다 정규표현식을 파라미터로 받아 내부적으로 Pattern 인스턴스로 컴파일하는것을  확인할 수 있다.

```java
static boolean isRomanNumeralSlow(String s) {
		return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
			+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");//293
	}
```

```java
public boolean matches(String regex) {
        return Pattern.matches(regex, this);
    }
```

```java
public static boolean matches(String regex, CharSequence input) {
        Pattern p = Pattern.compile(regex);
        Matcher m = p.matcher(input);
        return m.matches();
    }
```

필드로 선언하여 사용 객체 생성 시 불필요하게 드는 비용 감소

```java
import java.util.regex.Pattern;

/**
 * @author jhkim
 * @since 2022-11-29
 */
public class Test {
	static boolean isRomanNumeralSlow(String s) {
		return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
			+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");//293
	}

	private static final Pattern ROMAN = Pattern.compile(
		"^(?=.)M*(C[MD]|D?C{0,3})"
			+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeralFast(String s) {
		return ROMAN.matcher(s).matches();//62
	}

	public static void main(String[] args) {
		boolean b = false;

			long start = System.currentTimeMillis();
			for (int j = 0; j < 100000; j++) {
				b ^= isRomanNumeralFast("MCMLXXVI");  // Change Slow to Fast to see performance difference
			}
			long end = System.currentTimeMillis();
			System.out.println(end - start);
	}
}
```

### 예시3) AutoBoxing

- primitive type →  Wrapper 타입으로 변경 : AutoBoxing
- Wrapper 타입 → primitive type으로 변경  : UnBoxing
- 기본타입과 박싱된 Wrapper타입을 섞어서 사용하면 변환과정에서 **불필요한 객체가 생성 될 수 있다.**

```java
private static long sum() {
		Long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE ; i++) {
			sum += i;
		}
		return sum;
	}
	public static void main(String[] args) {
		long start = System.nanoTime();
		long x = sum();
		long end = System.nanoTime();
		System.out.println((end - start) / 1000000 + "ms");//7236ms
		System.out.println(x);

	}
```

```java
private static long sum() {
		long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE ; i++) {
			sum += i;
		}
		return sum;
	}
	public static void main(String[] args) {
		long start = System.nanoTime();
		long x = sum();
		long end = System.nanoTime();
		System.out.println((end - start) / 1000000 + "ms");//797ms
		System.out.println(x);

	}
```

---

참고
- https://coding-factory.tistory.com/536