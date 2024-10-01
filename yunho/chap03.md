# item10: equals는 일반 규약을 지켜 재정의하라

> equals 메서드는 재정의 하기 쉬어보이지만 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래한다.

## equals를 재정의하지 않아도 되는 경우

1. 각 인스턴스가 본질적으로 고유하다.
2. 인스턴의 논리적 도치성을 검사항 일이 없다.
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
4. 클래스가 private이거나 package-private이고 메서드를 호출할 일이 없다.

## 그러면 언제 equals를 재정의해야 할까?
-> equals는 논리적인 동치성을 확인하고 싶을 때 재정의 한다.

예외: Enum 값 클래스라고 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장할떄

## equals의 일반 규약
null이 아닌 모든 참조 값 x, y, x에 대하여

* 반사성: 
  * x.equals(x) == true
* 대칭성: 
  * x.equals(y) == true
  * y.equals(x) == true
* 추이성:
  * x.equals(y) == true
  * y.equals(z) == true
  * x.equals(z) == true
* 일관성:
  * x.equals(y)를 반복해도 true 또는 false만 나온다.
* null-아님:
  * x.equals(null) == false

## Object 명세에서 말하는 동치 관계란?
* 반사성
  * 객체는 자기 자신과 같아야 한다
* 대칭성
  * 두 객체는 서로에 대한 동치 여부에 똑같이 답해야한다.
```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
    
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

void symmetryTest() {
    CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
    String test = "test";
    System.out.println(caseInsensitiveString.equals(test)); // true
    System.out.println(test.equals(caseInsensitiveString)); // false
}

```
* 추이성
  * null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true이다.
```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}

public class ColorPoint extends Point {

  private final Color color;

  @Override
  public boolean equals(Object o) {
    if (!(o instanceof Point)) {
      return false;
    }

    // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
    if (!(o instanceof ColorPoint)) {
      return o.equals(this);
    }

    // o가 ColorPoint이면 색상까지 비교한다.
    return super.equals(o) && this.color == ((ColorPoint) o).color;
  }
}

void transitivityTest() {
  ColorPoint a = new ColorPoint(2, 3, Color.RED);
  Point b = new Point(2, 3);
  ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

  System.out.println(a.equals(b)); // true
  System.out.println(b.equals(c)); // true
  System.out.println(a.equals(c)); // false
}
```
* 추이성(무한 재귀)
```java
public class SmellPoint extends Point {

    private final Smell smell;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof SmellPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.smell == ((SmellPoint) o).smell;
    }
}

void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new SmellPoint(2, 3, Smell.SWEET);

    System.out.println(cp.equals(sp));
}
```
* 추이성(리스코프 치환 원칙)
```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}

```
이렇게 되면 동작은 하지만 리스코프 치환 원칙을 지키지 못한다.
* 추이성(상속대신 컴포지션을 사용해라)
```java
public class ColorPoint2 {

    private Point point;
    private Color color;

    public ColorPoint2(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```
* 추이성(추상 클래스)
  * 추상 클래스의 하위 클래스에서라면 equals의 규약을 지키면서도 값을 추가할 수 있다.

* 일관성
  * null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 할상 true를 반환하거나 항상 false를 반환한다.
  * java.net.URL 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교하기 때문에 같은 도메인 주소라도 나오는 IP 주소가 다르므르 반복적으로 호출한 경우 결과가 달라질 수 있다.

* null이 아님
  * null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false
```java
@Override
public boolean equals(Object o) {
  // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)거 null이면 false를 반환하기 때문이다. 
  if (!(o instanceof Point)) {
    return false;
  }
}
```
## equals 좋은 재정의 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연사자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환 한다.
4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
   5. float과 double을 제외한 기본 타입 필드는 ==를 사용한다ㅣ.
   6. 필드가 참조 타입이라면 equals를 사용한다.
   7. null 값을 정상 값으로 취급한다면 Objects.equals로 NullpointException을 예방하자
## 마무리
* 꼭 필요한 경우가 아니라면 재정의하지 말자.
* 그래도 필요하다면 핵심필드를 빠짐 없이 비교하며 다섯 가지 규약을 지키자.
* 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 더 싼 필드를 먼저 비교하자.
* 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.
* 핵심 필드로부터 파생되는 필드가 있다면 굳이 둘다 비교할 필요는 없다. 편한 쪽을 선택하자.
* equals를 재정의할 땐 hashCode도 반드시 재정의하자
* equals의 매개변수 입력을 Object가 아닌 타입으로는 선언하지 말자.
* @Override 어노테이션을 일관되게 사용하라

# item11: equals를 재정의하려거든 hashCode도 재정의하라
> equals를 재정의한 클래스는 hashCode도 재정의 해야 한다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.

## hashCode 일반 규약
* equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
(애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
* equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
* equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다
```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(010, 1234, 5678), new Person("윤호"));

map.get(new PhoneNumber(010, 1234, 5678)); 

```
다음과 같이 get을 하면 반환되는 값이 없다. (null)
## 동치인 모든 객체에서 똑같은 hashCode를 반환하는 코드
```java
@Override 
public int hashCode() {
    return 42;
}
```
* 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시 테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작한다.
* 그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 도저히 쓸 수 없게 된다.

## Objects.hash() 메서드
```java
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```
* 속도가 느리다.
* 내부적으로는 Objects.hash() 메서드가 인수들을 배열로 만들고, 이 배열을 이용하여 해시코드를 계산한다.
  * 객체를 감싸는 임시 객체로 박싱을 수행함
* 박싱과 언박싱 과정은 성능 저하를 가져올 수 있다.
  * 본 타입에서 해당 객체를 생성하거나, 객체를 기본 타입으로 변환하기 위해 불필요한 오버헤드가 발생하기 때문

## 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다
1. int 변수인 result를 선언한 후 값을 c로 초기화한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
```java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```

## 해시코드를 지연 초기화하는 hashCode 메서드
* 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 한다.
  * 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
* 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산한느 지연 초기화하면 좋다.
  *  필드를 지연 초기화 하려면 그 클래스가 thread-safe가 되도록 동기화에 신경 쓰는 것이 좋다.

```java
private int hashCode;

@Override
public int hashCode() {
      	int result = hashCode;  // 캐시된 해시코드 값을 가져옴
        if(result == 0) {  // 캐시된 값이 없다면 새로 계산
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
        }
        return result;
}
```
* 해시코드를 지연 초기화하는 hashCode 메서드

## 결론
hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클리언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수 있다.

# item12 : toString을 항상 재정의하다
## toString을 재정의해야 하는 이유
* toString()을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 해당 클래스를 사용한 시스템은 디버깅에 용이하다
* 직접 호출하지 않더라도 println, 문자열 연결, assert 구문, 디버거의 객체 출력 등에서 자주 쓰인다
* map 객체의 경우 {Jenny = PhoneNumber@addbb}보다 {Jenny = 012-1234-5678}라는 메시지가 훨씬 가독성이 높다

## 그럼 어떻게 재정의 하는 게 좋을까?
1. 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
2. 포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.
3. toString()이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자
## toString을 재정의 하지 않아도 되는 경우
* 정적 유틸리티 클래스
* 이미 toString이 제공되는 Enum타입

# item13: clone 재정의는 주의해서 진행하라
* 낯선 개념이여서 어렵다.
* 간단하게 말하면 Cloneable을 implement한 클래스는 복제 가능 여부를 알려준다.
* 기본적으로 얕은 복사를 한다
## clone의 규약
1. x.clone() != x
   → 반드시 true

2. x.clone().getClass() == x.getClass()
   → 반드시 true

3. x.clone().equals(x)
   → true일 수도 있고 아닐 수도 있다.

4. x.clone.getClass() == x.getClass()
* 이외 많은 내용들이 존재하지만 결론은 이거다
  * 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하는게 최고다 
    * TreeSet()의 생성자를 찾아보자
  * 단 배열만은 clone 메서드 방식이 가장 깔끔하기 때문에 위 규칙에 예외다.
    * 얕은 복사로 충분한 경우가 많고, 기본 자료형 배열은 사실상 깊은 복사처럼 동작하기 때문에

# item14 : Comparable을 구현할지 고려하라
>Comparable 인터페이스를 구현한 클래스의 인스턴스는 자연적인 순서를 가지게 된다.
Comparable 인터페이스는 compareTo라는 메서드를 하나 가진다. compareTo 메서드는 동치성 비교와 순서 비교가 가능하다.
자바의 거의 모든 값 클래스와 열거 타입이 Comparable을 구현한 상태이다.
```java
public interface Comparable<T> {
public int compareTo(T o);
}
```
* Comparable의 compareTo는 동치성뿐만 아니라 순서까지 비교 가능하다
* 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현한다
*  알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자
## compareTo 메서드의 일반 규약
1. 두 객체의 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 함.
2. 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 함.
3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 함.
4. (필수X) compareTo 메서드로 수행한 동치성 테스트 결과가 equals의 결과와 같야아 함.
   * 지키지 않을 경우, 컬렉션이 구현한 인터페이스 (Collection, Set, Map 등) 에 정의된 동작과 다르게 작동할 수 있음.
   * 정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용하기 때문.
   
**Comparable 주의사항**

* 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 수 없음.
* Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶을 경우, 확장하는 대신 독립된 클래스를 만들고, 해당 클래스에 원 클래스의 인스턴스를 가리키는 필드를 두는 방식으로 처리하고, 내부 인스턴스를 반환하는 뷰 메서드를 제공하는 형태로 처리해야 함.
## compareTo 메서드 작성 요령
* Comparable은 제네릭 인터페이스이므로 타입을 확인하거나 형변환할 필요 없음.
* 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출 필요.
* Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator를 대신 사용.
* 정수 기본 타입, 실수 기본 타입을 비교할 때는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용.
* 클래스에 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교.
## Comparator
* 자바 8에서는 Comparator 인터페이스가 비교자 생성 메서드를 제공하여 메서드 연쇄 방식으로 비교자를 생성할 수 있음.
* 이를 활용하여 Comparable 인터페이스의 compareTo 메서드를 구현 가능.

