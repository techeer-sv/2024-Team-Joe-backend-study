# item01

## 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩토리 메서드의 장점

**1. 이름을 가질 수 있다**

```java
Person person = new Person("정윤호", 26, "korean");
// 한국인이라는 사람객체를 생성시 명시적으로 메서드 명에 나타내줄 수 있음
Person person = Person.korean("정윤호", 26);
```

**2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**

* 반복되는 요청에 같은 객체를 반환하는 식으로 언제 어느 인스턴스를 살아 할지를 철저히 통제할 수 있다.
* 인스턴스를 통제하는 이유는?
* 싱글턴, 단일 인스턴스 보장
  **3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

```java
public interface Person {


}

public class Korean implements Person {

  int age = 0;

  public static Person create(int age) {
    return new Korean(age);
  }

}
```

"API를 만들때 이 유연성을 응요하면 구현 클래스를 공개하지 않고도 그 객체를 반환 할 수 있어 API를 작게 유지할 수 이다."

-> 구현 클래스에 의존하지 않고 객체를 반환가능 하다.

-> 구현 클래스를 고정하지 않고, 따라서 코드를 수정하지 않고 다른 구현체를 쓸 수 있음, 결과적으로 API를 작게 유지 가능(일일이 메서드를 선언 안해도 된다.)

**4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다.**
3번에서 Person을 implement 받는 경우라고 생각하자. 반환값의 Type을 Person으로 해두면 값을 유연하게 가져갈 수 있다.

**5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

-> 유연함이 프레임워크를 만드는 근간이 됨

간단하게 "동작을 정의하는 서비스 인터페이스", "구현체를 사용하는 제공자 등록 API", "서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API"

JDBC에 대입하면

* Connection는 서비스 인터페이스
* DriverManger.registerDriver는 제공자 등록 API
* DriverManger.getConnection은 서비스 접근 API
* Driver가 서비스 제공자 인터페이스 역할

### 정적펙토리 메서드 단점

**1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.**

```java.utils.Math```를 생각하면 좋을 듯 (상속이 불가하다)

**2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다**

생성자는 Javadoc에 보면 자동으로 상단에 위치한다.

# item02

## 생성자에 매개변수가 많다면 빌더를 고려하라

### 생성자 패턴

```java
// 다음과 같은 클래스가 있다고 가정하자
public class User {

  private final int age;             // 필수
  private final int phoneNumber;     // 필수
  private final int weight;          // 선택
  private final int tall;            // 선택
  private final int birthday;        // 선택
}

//점층적 생성자 패던으로 생성을 하면 
private User(int age, int phoneNumber) {
  this.age = age;
  this.phoneNumber = phoneNumber;
}

private User(int age, int phoneNumber, int weight) {
  this(age, phoneNumber);
  this.weight = weight;
}

private User(int age, int phoneNumber, int weight, int tall) {
  this(age, phoneNumber);
  this.weight = weight;
  this.tall = tall;
}

private User(int age, int phoneNumber, int weight, int tall, int birthday) {
  this(age, phoneNumber);
  this.weight = weight;
  this.tall = tall;
  this.birthday = birthday;
}

User user = new User(20, 99998888, 70, 180, 1225);
```

매개변수가 늘어날수록 코드를 작성하거나 읽기 어렵다.

* 값이 무엇인지 헷갈리고
* 컴파일러는 잘못된 값을 알아차리지 못한다

### 자바빈즈 패턴

```java
// 자바빈즈 패턴
public class User {

  private int age = 1;
  private int phoneNumber = 11111111;
  private int weight;
  private int tall;
  private int birthday;

  public void setAge(final int age) {
    this.age = age;
  }

  public void setPhoneNumber(final int phoneNumber) {
    this.phoneNumber = phoneNumber;
  }

  public void setWeight(final int weight) {
    this.weight = weight;
  }

  public void setTall(final int tall) {
    this.tall = tall;
  }

  public void setBirthday(final int birthday) {
    this.birthday = birthday;
  }
}

User user = new User();
user.

setAge(20);
user.

setPhoneNumber(99999999);
user.

setWeight(70);
user.

setTall(180);
user.

setBirthday(1225);
```

어떤 값을 초기화 하는지 명시적으로 보임

* 하지만 다음과 같은 단점이 존재
    * 객체 하나에 여러 메서드를 호출해야 함
    * 객체가 완전히 생성되기 전까지 일관성이 무너짐
    * 일관성이 깨지면 버그가 존재하고 해당 버그 때문에 디버깅이 힘들어 진다.

### 빌더 패턴

```java
// 점층적 생성자 패턴의 안전성 + 자바 빈즈의 가독성을 가져감
public class User {

  private final int age;
  private final int phoneNumber;
  private int weight;
  private int tall;
  private int birthday;

  public User(Builder builder) {
    this.age = builder.age;
    this.phoneNumber = builder.phoneNumber;
    this.weight = builder.weight;
    this.tall = builder.tall;
    this.birthday = builder.birthday;
  }

  public static class Builder {

    private final int age;
    private final int phoneNumber;
    private int weight;
    private int tall;
    private int birthday;

    public Builder(int age, int phoneNumber) {
      this.age = age;
      this.phoneNumber = phoneNumber;
    }

    public Builder weight(int weight) {
      // validation 가능
      this.weight = weight;
      return this;
    }

    public Builder tall(int tall) {
      this.tall = tall;
      return this;
    }

    public Builder birthday(int birthday) {
      this.birthday = birthday;
      return this;
    }

    public User build() {
      return new User(this);
    }
  }
}

User user = new User.Builder(20, 99998888)
    .weight(70)
    .tall(180)
    .birthday(1225)
    .build();
```

결과적으로 User 클래스는 불변하게되었고, 어떤값을 초기화 하는지 알 수 있다.

chaining method로 객체를 생성할 수 있다.

빌터 패턴은 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있다.

하지만 장점만 있는 건 아니다. 빌더 메서드를 만드는데 리소스가 필요하고, 매개변수는 4개는 넘어야 값어치가 있다.

# item03

## private 생성자나 열거 타입으로 싱글턴을 보증하라

### 싱글턴이란?

인스턴스를 오직 하나만 생성할 수 있는 클래스

### 싱글던을 만드는 방식

```java
// public static final 필드 방식의 싱글턴
public class Game {

  private static final Game INSTANCE = new Game();

  private Game() {
  }
}
```

private 생성자를 통해 Game.INSTANCE를 초기화할 때 단 한번만 호출된다.

* 장점
    * 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
    * 필드를 final로 선언해 다른 객체를 참조할 수 없음
    * 간결함
* 단점
    * 클라이언트에서 사용하지 않더라고 인스턴스가 항상 생성됨

```java
// 정적 팩터리 방식의 싱글턴
public class Game {

  private static final Game INSTANCE = new Game();

  private Game() { ...}

  public static Game getInstance() {
    return INSTANCE;
  }
}
```

* Game 인스턴스는 단 한번만 만들어짐
    * 리플렉션을 통한 예외는 적용됨
* 장점
    * API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음
    * 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있음
    * 원한다면 정적 팩터리를 제너릭 싱글턴 팩터리로 만들 수 있음
    * 정적 펙터리의 메서드 참조 공급자를 사용할 수 있음
* 단점
    * 사용하지 않아도 인스턴스가 생성됨

```java
// enum 타입 방식의 싱글턴
public enum Game {
  INSTANCE;
}
```

* public 필드 방식과 비슷하지만 더 간결함
* 추가 노력 없이 직렬화할 수 있음
    * 심지어 아주 복잡한 직렬화 상황이나 리프렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아줌
* 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임
* 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없음
* 열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있음

# item04

## 인스턴스화를 막기 위해 private 생성자를 사용하라

### 1. 기본 타입 값이나 배열 관련 메서드를 모은 클래스

```java
public final class Math {

  /**
   * Don't let anyone instantiate this class.
   */
  private Math() {
  }
}
```

Math는 util성 클래스이므로 Math를 직접 선언하지 않는다.
(private으로 생성자가 막혀있는 것을 볼 수 있다)

### 2. 특정 인터페이스를 구현하는 객체를 생성해주는 정적 펙토리를 모은 클래스

```java.util.Collections```를 보면 private 생성자 및 static으로 구현 되어있는 상태다

### 추상 클래스로는 인스턴스화를 막을 수 없다.

* 하위 클래스를 인스턴스화 해서 사용할 수 있음
* 상속해서 쓰라는 뜻으로 오해할 수 있다.

# item05

## 자원을 직접 명시하지 말고 의존 객체를 주입을 사용하라

최근 Redis의 TTL을 검사하는 로직을 utils 메서드로 작성을 한 경험이 있다.

```java

@UtilityClass
public class RedisTtlCalculator {

  public static long stayQrSeedTtlSec(LocalDateTime checkoutDatetime) {
    // checkoutDatetime - LocalDateTime.now() + bonusTtlMin
    return Duration.between(
            LocalDateTime.now(), checkoutDatetime.plusMinutes(QrType.STAY.getBonusTtlMin()))
        .getSeconds();
  }
}

```

그런 다음과 같은 문제가 발생했다.

* test를 작성시 LocalDateTime.now()를 기준으로 하다 보니 고정된 값을 mock데이터로 못넣음
* static method는 mockito를 사용못함

책에 나오는 내용을 보면

**사용하는 자원에 따라 동작이 달라 지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식은 적합하지 않다**

-> 따라서 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식인 **의존 주입**으로 구현을 하자

### 의존 객체 주입

```java
public class SomeService {

  private final TTLCalculator ttlCalculator;
  
  ...

}
```

### 의존성 주입의 장점

TTL 을 검증하는 로직을 테스트하기 위해서는 LocalTimeDate.now()를 활용해야한다.

하지만 가변적인 값을 사용할 수 없으므로 새로운 test용 mock객체를 만들어 test에서 원하는 값을 집어 넣을 수 있다.

### 장점

위와 같이 의존성을 주입해주는 방법은 객체의 유연성을 늘려주고 객체간의 결합도를 낮출 수 있는 효과를 가지고 있다.

# item06

## 불필요한 객체 생성을 피하라

```java
// 선언을 할 때 마다 heap 메모리에 해당 값이 계속 저장이 됨
String text = new String("text");
// 다음과 같이 선언을 하면 StringPool에서 값을 가져와 재활용
String text = "text";

// 마찬가지로 Boolean도 그 예시가 될 수 있다.
Boolean(String); // X
Boolean.valueOf(String); // O
// Boolean도 미리 선언된 객체를 재활용 하는 식으로 사용을하므로 정적펙터리 메서드로 객체를 선언하자!

// Worst case
// Pattern 인스턴스를 만들어 사용하는데 Pattern 인스턴스는 입력받은 정규표현식의 유한 상태 머신을 만들기 때문에 생성 비용이 높다.
// 이런 생성 비용이 높은 Pattern 인스턴스를 한 번 쓰고 버리는 구조로 만들어 곧바로 GC의 대상이 되게 만들고 있다. 즉, 비싼 객체라고 할 수 있다.
static boolean isNumeric(String str) {
  return str.matches("[0-9]]");
}
// Best case
// 이런 문제 때문에 Pattern 객체를 미리 만들어 컴파일하고 재사용을 하는 것이 좋다
public class Number {
  private static final Pattern NUMERIC = Pattern.compile("[0-9]]");

  static boolean isNumeric(String str) {
    return ROMAN.matcher(str).matches();
  }
}

// Auto Boxing도 조심해야한다.
// Wrapper 타입의 경우 2^31 만큼의 Long을 heap 메모리에 저장을 하는 행위를 하고 있다.
// 그에 반해 primitive 타입은 stack 메모리에 저장 되어서 매우 빠른 속도로 값을 가져올 수 있습니다.
//  걸린 시간: 6.3s
void autoBoxing_Test() {
  Long sum = 0L;
  for(long i = 0; i <= Integer.MAX_VALUE; i++) {
    sum += i;
  }
}
// 걸린 시간: 0.59s
void noneBoxing_Test() {
  long sum = 0L;
  for(long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
  }
}
```


# item07

## 다 쓴 객체 참조를 해제하라
### Stack
자바에서 GC(Garbage Collector)가 있다고 해서 메모리 관리를 신경 안써도 되는 건 아니다.

다음은 Stack 자료구조이다.
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 갖고 있기 때문이다. elements 배열의 활성 영역(인덱스가 size 보다 작은 곳) 밖의 참조들을 가리킨다.

가비지 컬렉션 언어에서는 메모리 누수를 찾기 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체를 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체..)를 회수해가지 못한다.

여기서는 Stack 자체가 정리되지 못한다. Stack이 갖고 있는 Object[] elements가 가지고 있는 요소들이 정리되지 못하기 때문이다.

그럼 방법은 뭐냐: 다 쓴 참조는 null을 할당하자(GC에서 알아서 가져감)

```java
public class Stack {
	...

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
}
```

### 캐시
* 캐시 또한 메모리 누수의 주범이다
  * 객체 참졸르 캐시에 넣어두고 잊어버리면 메모리를 계속 차지하고 있는 상태다
  * `WeakHashMap`을 활용하면 된다
    * 다 쓴 엔트리를 즉시 자동으로 제거해준다

**메모리 누수는 겉으로 잘 안들어난다. 따라서 미리 예방이 중요하다!!**

# item08

## finalizer와 cleaner 사용을 피하라

* finalizer는 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
  * (gpt에게 물어보니 finalizer를 사용한 구현체는 이미 다 deprecated 되었다고 한다.)
* cleaner는 finalizer보다는 덜 위험하지만, 여전히 에측할 수 없고 느리고, 일반적으로 불필요하다.
* 둘 다 신속히 수행할지는 전적으로 가비지 컬렉터에 달렸다. 따라서 가비지 컬렉터의 성능에 따라 종료 시간이 천차만별이다.
* 수행 여부를 보장하지 않는다.
  * 즉 종료 작업을 수행하지 못한채 중단될 수 있다는 말이다.
  * 따라서 상태를 영구적으로 수정하는 작업에서는 절대 사용하지 말아야한다.
* 둘다 심각한 성능 문제도 동반한다.
* finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.
  * 생성자나 직렬화 과정에서 예외가 발생하면 하위 클래스의 finalizer가 수행되는데 악의적인 작성된 경우 영향을 미친다.
  * 따라서 finaldㅣ 아닌 클래스를 finalizer 공격으로 부터 방어하려면 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하자
  * AutoCloseable를 사용하자 (item09를 확인)

# item09

## try-finally 보다는 try-with-resources를 사용하라
자바 라이브러리에는 close() 메서드를 호출해 직접 닫아줘야 하는 자원이 많다(ex. InputStream, OutputStream, java.sql.Connection)

-> 즉 직접 닫아주는 것을 잊으면 자원 관리 문제로 이어진다

과거에는 try-finally로 닫힘을 보장하는 수단으로 쓰였다
```java
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
하지만 만약 자원을 하나 더 사용한다면 어떻게 될까?

```java
import java.io.FileOutputStream;

public static void inputAndWriteString() throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0) {
        out.write(buf, 0, n);
      }
    } finally {
      out.close();
    }
  } finally {
    in.close();
  }
}
```
예외는 try블록과 finally 블록 모두 두 코드 에제에조차 미묘한 결점이 있다.

에외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다.

이러한 문제를 해결하기위해 `try-with-resources`를 사용한다.

```java
import java.io.FileInputStream;
import java.io.IOException;

static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src)) {
    OutputStream out = new FIleOutputStream(dst) {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while((n =in.read(buf))>=0)
        out.write(buf, 0, n);
    }
  }
}
```
가독성이 좋아졌다. 

또한 가독성 뿐만이 예외가 발생했을 때 디버깅 하기에도 더 편리하다. OutputStream(내부에서 자원을 사용하는 객체)에서 예외가 발생하는 경우, close(물론 코드 상으로는 보이지 않지만) 호출 시 발생하는 예외는 숨겨지고 write() 예외가 기록된다.

