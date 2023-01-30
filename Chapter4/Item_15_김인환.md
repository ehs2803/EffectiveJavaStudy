## 클래스와 멤버의 접근 권한을 최소화하라

잘 설계된 컴포넌트는 외부로부터 내부 구현이 잘 숨겨져 있어서 구현과 API가 깔끔히 분리되어 있다. 
이것을 정보은닉, 또는 캡슐화라고 한다.

### 정보 은닉의 장점

- 시스템 개발 속도 향상
- 시스템 관리 비용 절감
- 성능 최적화에 도움을 줌
- 소프트웨어 재사용성 향상
- 큰 시스템 제작의 난이도 감소


### 접근 제한 수준

- private : 멤버를 선언한 톱레벨 클래스만 접근 가능
- package-private(default) : 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
- protected : package-private의 접근 범위 포함, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능
- public : 모든 곳에서 접근 가능

### 접근 제한자 활용 원칙

**모든 클래스와 멤버의 접근성을 가능한 한 좁힌다.**

- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다
  - final이 아닌 필드를 public 으로 선언할 경우에는, 해당 필드에 담는 값을 제한할 수 없게 되기 때문
  - 필드가 수정될 때 다른 작업을 할 수 없게 되어서 스레드 안전하지 않기 때문
  - 해당 클래스의 추상 개념을 완성하는 데 꼭 필요한 상수라면 public static final로 공개할 수 있음

- 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다
  - 길이가 0이 아닌 배열은 변경이 가능하기 때문
  - 잘못된 예
    ```java
    public static final Thing[] VALUES = { ... };
    ```
  - 
    ```java
    // 1 : 배열을 private으로 만들고 public 불변 리스트를 추가하는 방식
    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final List<Thing> VALUES = 
        Collections.unmodifiableList(Arrays.asList(PRIVAZTE_VALUES));
    
    // 2 : 배열을 private으로 만들고 복사본을 반환하는 방식
    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final Thing[] values() {
        return PRIVATE_VALUES.clone();
    }
    ```

### 핵심 정리
- 프로그램 요소의 접근성은 가능한 한 최소한으로
- public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져선 안됨
  - public static final 필드가 참조하는 객체가 불변인지 확인하라