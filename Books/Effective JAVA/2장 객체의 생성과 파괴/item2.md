## item-2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 1. 점층적 생성자 패턴 (Telescoping Constructor Pattern)
- 생성자 체이닝 방식
- 기존에 선언한 생성자를 재사용한다.

### 2. 자바빈즈 패턴(JavaBeans Pattern)
- 기본 생성자로 객체 생성
- setter 메서드로 필드 값 할당

**단점**
- 일관성이 깨진다.
- 불변 객체로 만들기 어렵다.

### 3. 빌더 패턴 (Builder Pattern)
- 직접 객체를 만들지 않고, 정적 팩토리(또는 생성자)를 호출해 빌더 객체를 얻는다.
- 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어 둔다.

#### 계층형 빌더
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.
> - self() 를 사용하면 형변환을 줄일 수 있다. 
> - 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza {
  public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
  final Set<Topping> toppings;
  
  abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }
    
    abstract Pizza build();
    
    // 하위 클래스는 이 메서드를 재정의(overriding)하여
    // "this"를 반환하도록 해야 한다.
    protected abstract T self();
  }
  
  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // 아이템 50 참조
  }
}
```

- 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.
- 이렇게 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 `공변환 타이핑` 이라 한다.

```java
public class NyPizza extends Pizza { 
  public enum Size { SMALL, MEDIUM, LARGE }
  private final Size size;
  
  public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;
    
	// 피자 크기는 필수 매개변수
    public Builder(Size size) {
      this.size = Objects.requireNonNull(size);
    }
	
    @Override 
    public NyPizza build() {
      return new NyPizza(this);
    }
	
    @Override 
    protected Builder self() { 
		return this; 
	}
  }
  
  private NyPizza(Builder builder) {
    super(builder);
    size = builder.size;
  }
}
```

**장점**
- 객체 생성에 유연하다.
    - 클라이언트에서 필요한 필드의 모든 경우를 선택적으로 생성 가능하다.
- 확장성이 뛰어나다.
    - API는 시간이 지날수록 파라미터가 많아지기 때문에 확장성이 중요하다.

**단점**
- 필수 값을 지정할 수 없다.
