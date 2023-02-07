# item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
인터페이스의 사용 용도

- 인터페이스 구현체에게 무엇을 구현할지 알려주는 역할
    - 위의 역할로만 사용되길 권장함
- 안티패턴(소프트웨어 공학에서 실제로 많이 사용되지만 가독성, 성능, 유지보수 측면에서 지양하는 패턴)
    - 상수 인터페이스
    - 메서드 없고, static final 필드만 존재 → 상수 인터페이스 안티패턴

    ```java
    public interface PhysicalConstants {
    	static final double AVOGARDROS_NUMBER1 = 6.022;
    	static final double AVOGARDROS_NUMBER2 = 6.0222949;
    	static final double AVOGARDROS_NUMBER3 = 6.02241122;
    
    }
    ```

    ```java
    public class PhysicalConstantsImpl implements PhysicalConstants{
    	public static void main(String[] args) {
    		System.out.println("AVOGARDROS_NUMBER1 = " + AVOGARDROS_NUMBER1);
    	}
    }
    ```

- 상수 공개목적 하위 방식 적절
    1. 해당 클래스와 연관된 직접된 상수라면 클래스 내부에 열거타입으로 구현
    2. 인스턴스화 할 수 없는 유틸리티 클래스로 생성

    ```java
    public class PhysicalConstantsUtil {
    	private PhysicalConstantsUtil(){}//인스턴스화 방지
    	static final double AVOGARDROS_NUMBER1 = 6.022;
    	static final double AVOGARDROS_NUMBER2 = 6.0222949;
    	static final double AVOGARDROS_NUMBER3 = 6.02241122;
    
    }
    ```

    ```java
    package org.example;
    
    import static org.example.PhysicalConstantsUtil.*;
    
    /**
     * @author jhkim
     * @since 2023/02/06
     *
     */
    public class PhysicalConstantsImpl {
    	public static void main(String[] args) {
    		System.out.println("AVOGARDROS_NUMBER1 = " + AVOGARDROS_NUMBER1);
    	}
    }
    ```