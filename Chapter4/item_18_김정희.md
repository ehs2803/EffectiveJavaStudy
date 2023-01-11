클래스가 다른 클래스를 구현 상속일 경우 상속은 캡슐화를 깨지게 한다. 이의 경우 상속에의해 캡슐화가 깨지게 된다.

상위클래스의 구현의 변경에 따라 하위클래스에 여러 영향을 줄 수 있다.

아래 예제는 HashSet을 상속받아 add, addAll을 재정의한 예제이다. 추가된 요소들의 개수를 확인할 수 있는 메서드이다.

```java
/**
 * @author jhkim
 * @since 2023/01/07
 *
 */
public class InstrumentHashSet<E> extends HashSet<E> {
	private int addCount = 0;

	public InstrumentHashSet() {

	}
	public InstrumentHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}

	public static void main(String[] args) {
		InstrumentHashSet<String> s = new InstrumentHashSet<>();
		s.addAll(List.of("a","ab","abc"));
		System.out.println("s.getAddCount() = " + s.getAddCount());// 6
	}
}
```

예상 결과인 3과는 다르게 6이 출력된다. addAll메서드에 정의된 상위 클래스의 addAll 메서드안에서 add메서드는 재 구현된 add 메서드를 호출하게된다. 
이러한 경우 상위 클래스의 변경에 하위 클래스에 영향을 주게 된다.

![image](https://user-images.githubusercontent.com/45592236/211807250-e20483dd-7df3-411a-9f97-917c2c63280d.png)

이를 방지하기 위해 Composition(새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는방식)을 책에서 제안한다.

- Composition
    - 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는방식
- Forwarding
    - 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환