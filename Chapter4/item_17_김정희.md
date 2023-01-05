# item17.변경 가능성을 최소화 하라 
## 불변 클래스

인스턴스가 소멸될때까지 인스턴스가 가지고 있는 내부의 값들이 변경되지 않는 인스턴스

불변 클래스 생성 방법

1. 객체의 상태를 변경하는 메서드 제공하지 않는다. → setter메서드 제공 x
2. 클래스를 확장할 수 있도록 한다. → 상속을 하지 못하도록 final클래스로 선언

    ```java
    public class SamsungCalculator extends CommonCalculate{
    	public SamsungCalculator(short num, short operator) {
    		super(num, operator);
    	}
    	private String name;
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    }
    ```

3. 모든 필드를 final로 선언한다.
    1. 성능적 장점때문에 쓸 수 있다면 많이 쓰는것이 좋다.
4. 모든 필드는 private으로 선언한다.
5. 자신 외에는 내부의 가변 컴포넌트에 접근 할 수 없다.

    ```java
    public final class Person {
    
    	private final Address address;//가변적 클래스
    
    	public Person(Address address) {
    		this.address = address;
    	}
    
    	public Address getAddress() {
    		return address;
    	}
    	
    	public static void main(String[] args) {
    			Address seattle = new Address();
    			seattle.setStreet("street1");		
    			Person person = new Person(seattle);
    			
    			Address redmond = person.getAddress();
    			redmond.setStreet("street2");
    			System.out.println("person.getAddress().getStreet() = " + person.getAddress().getStreet());
    	}
    		}
    }
    ```

    ```java
    public class Address {
    
    	private String zipCode;
    	private String street;
    
    	public String getZipCode() {
    		return zipCode;
    	}
    
    	public void setZipCode(String zipCode) {
    		this.zipCode = zipCode;
    	}
    
    	public String getStreet() {
    		return street;
    	}
    
    	public void setStreet(String street) {
    		this.street = street;
    	}
    }
    ```

   Person클래스가 불변 클래스더라도 Address가 가변클래스 이므로 Person클래스가 가지고있는 내부 필드가 얼마든지 변경 가능하게 됨

    ```java
    person.getAddress().getStreet() = street2
    ```

    ```java
    public Address getAddress() {
    		Address copyOfAddress = new Address();
    		copyOfAddress.setStreet(address.getStreet());
    		copyOfAddress.setZipCode(address.getStreet());
    		return copyOfAddress;
    }
    ```

   그러므로 Address에 접근하는 객체를 바로 리턴하는것이 아닌 방어적인 복사를 해서 객체의 값을 유지하도록 해야한다.