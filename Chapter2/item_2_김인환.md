## 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기가 어려운 단점이 있다.

### 점층적 생성자 패턴

- 필수 매개변수만 받는 생성자 부터 시작해 선택 매개변수를 전부 다 받는 생성자까지 매개변수를 늘려나가는 방식
- 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출
    - 이 경우, 보통 사용자가 설정하길 원하지 않는 매개변수 까지 포함할 수도 있는데, 어쩔 수 없이 그런 매개변수도 값을 지정해줘야 함.

즉, 점층적 생성자 패턴은 매개변수의 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵게 된다.

```java
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }

}
```

### 자바빈즈 패턴

- 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식
- 객체 하나를 만들기 위해 메서드를 여러개 호출해야 함
- 객체가 완전히 생성되기 전까지 일관성이 무너진 상태에 놓임
    - 이 때문에, 자바 빈즈 패턴으로는 클래스를 불변으로 만들 수 없음
    - 또한, 스레드 안정성을 위해 프로그래머가 추가 작업을 해주어야함

```java
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {
    }

    // Setters
    public void setServingSize(int val) {
        servingSize = val;
    }

    public void setServings(int val) {
        servings = val;
    }

    public void setCalories(int val) {
        calories = val;
    }

    public void setFat(int val) {
        fat = val;
    }

    public void setSodium(int val) {
        sodium = val;
    }

    public void setCarbohydrate(int val) {
        carbohydrate = val;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

### 빌더 패턴

- 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 모두 취한 패턴
- 빌더 패턴 사용 방법
  - 먼저, 필수 매개변수만으로 생성자를 호출해 빌더 객체 생성
  - 빌더 객체의 세터메서드를 활용해 선택 매개변수들을 설정
  - 이 후 build 메서드 호출로 원래 원하던 객체 생성
- 메소드 체이닝이 가능

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 빌더 자신을 반환하기 때문에 연쇄적인 호출이 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

- 계층적으로 설계된 클래스에 함께 쓰기 용이

```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Subclasses must override this method to return "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}

// Subclass with hierarchical builder (Page 15)
public class NyPizza extends Pizza {
  public enum Size { SMALL, MEDIUM, LARGE }
  private final Size size;

  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;

    public Builder(Size size) {
      this.size = Objects.requireNonNull(size);
    }

    @Override public NyPizza build() {
      return new NyPizza(this);
    }

    @Override protected Builder self() { return this; }
  }

  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }

  @Override public String toString() {
    return "New York Pizza with " + toppings;
  }
}
```


- '계층적 빌더' 사용 예시
```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```
### 빌더 패턴을 사용할 때 알아두면 좋은 점

- 객체를 만들기 전에 빌더부터 만들어야 함
    - 성능에 민감한 상황에서는 문제가 될 수 있음
- 점층적 생성자 패턴보다는 코드가 길어짐
    - 매개변수가 4개 이상은 되어야 가치가 있는 편
- API는 시간이 지날수록 매개변수가 많아지는 경향이 있음
    - 애초에 빌더로 시작하는 것이 좋은 편