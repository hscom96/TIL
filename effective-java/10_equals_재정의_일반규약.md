# 아이템 10. equals는 일반 규약을 지켜 재정의하라  



> 논리적 동치성 비교 가능하고, Map의 키와 Set의 원소로 사용 가능. 등등

<br/>

> ## 모든 객체의 공통 메서드
> Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)는  
> 모두 재정의(overriding)을 염두해 두고 설계된 것이라 재정의시 지켜야할 일반 규약이 있다.  
> __즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의 해야한다.__  
> 잘못 정의하면 오동작하게 될 수 도있음.(HashMap, HashSet ...)  

<br/>

## equals 재정의 안해도 괜찮은 경우는?
  1. 각 인스턴스가 본질적으로 고유한 경우.
      - ex) enum(아이템 34), 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(아이템1)
  2. 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없는 경우
  3. 상위 클래스에서 재정의한 equal가 하위 클래스에도 딱 들어 맞는 경우
  4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 경우

<br/>

## equals 일반 규약

### 1. 반사성 (reflexivity)
- x.equals(x)는 true (null 아닌 모든 x)
  - 즉, 자기 자신과 같아야한다.

<br/>

### 2. 대칭성 (symmetry)
- 조건
  - x.eqauls(y) true면 y.equals(x)도 true (null 아닌 모든 x,y)
  - 즉, 서로에 대한 동치가 같아야한다.  

#### 잘못된 코드
```java
// 잘못된 코드
public class CaseInsensitive {
  private final String s;
  @Override
  public boolean equals(Object o) {
	 if(o instanceof CaseInsensitiveString)
		  return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
	 if (o instanceof String) // 한 방향으로만 작동
		  return s.equalsIgnoreCase((String) o);
	 return false;
  }
  // 나머지 구현 생략
}
```
```java
CaseInsensitive cis = new CaseInsensitive("hyeonsu");
String s = "hyeonsu";
```
- cis.equals(s)는 true여도 s.equals(cis)는 false여서 대칭성 위배
- String의 equals는 CaseInsensitive를 모른다는 문제.

#### 정상 코드
```java
// 정상 코드
@Override
public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString &&
			((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

<br/>

### 3. 추이성 (transitivity)
- 조건
  - x.equals(y)가 true, y.equals(z)가 true면 x.equals(z)도 true (null 아닌 모든 x,y)

```java
public class Point {
    private final int x;
    private final int y;
    @Override
    public boolean equals(Object o) {
      // 구현 생략
    }
    // 나머지 코드 생략
  }
```

#### __문제 코드 #1 (대칭성 위배)__
```java
public class ColorPoint extends Point {
    private final Color color;
    @Override
    public boolean equals(Object o) {
      if (!(o instanceof ColorPoint))
          return false;
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
    // 나머지 코드 생략
  }
```
```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```
- p.equals(cp)는 true, cp.equals(p)는 false이므로 대칭성 위배한다.

#### __문제 코드 #2 (추이성 위배)__

```java
public class ColorPoint extends Point {
    private final Color color;
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        // o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        // o가 ColorPoint면 색상까지 비교한다.
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
    // 나머지 코드 생략
  }
```
  ```java
  ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
  Point p2 = new Point(1, 2);
  ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
  ```
  - 대칭성은 만족하지만 __추이성에 위배된다.__
    - p1.equals(p2), p2.equals(p3)는 true인데 p1.eqauls(p3)는 false
#### __즉, 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.__
#### __문제 코드 #3 (리스코프 치환 원칙 위배)__
  - 그렇다면 equals는 같은 구현 클래스의 객체와 비교할 때만 true 반환하게 변경하면?
    - instanceof 메서드가 아닌 getClass 메서드 사용
  ```java
  // Point 클래스 일부
    @Override public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
  ```
  - __하지만 리스코프 치환 원칙에 위배__
    - Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야한다.
> - 리스토프 치환 원칙이란?
>   - 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.  
>   - 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야한다.  

#### 우회 방법
  - __상속대신 컴포지션을 사용하자 (아이템 18)__
    - Point 상속대신 private 필드
    - ColorPoint와 같은 위치의 일반 Poinst를 반환하는 view메서드(아이템 6) 추가
  ```java
  public class ColorPoint {
      private final Point point;
      private final Color color;

      // 이 ColorPoint의 Point 뷰를 반환한다.
      public Point asPoint() {
          return point;
      }

      @Override public boolean equals(Object o) {
          if (!(o instanceof ColorPoint))
              return false;
          ColorPoint cp = (ColorPoint) o;
          return cp.point.equals(point) && cp.color.equals(color);
      }
      // 나머지 코드 생략
  }
  ```

<br/>

### 4. 일관성 (consistency)
- 조건
  - x.equals(y)를 반복해서 호출해도 항상 true (null이 아닌 모든 x, y)
    - 즉, 두 객체가 같다면 앞으로도 영원히 같아야한다. (객체가 수정되지 않는한)


<br/>

### 5. null 아님
- 조건
  - x.equals(null)은 false (null이 아닌 모든 x)
  - 즉, 모든 객체가 null과 같지 않아야 한다.
- 주의 사항
  - NullPointerException도 발생하는 것도 허용 안한다. (null 체크 검사)
  - 입력 매개 변수가 올바른 타입인지 검사하고 형변환 하자.
    - 잘못된 타입으로 인한 ClassCastException은 일반규약 위배됨

```java
// 올바른 타입 체크
@Override public boolean equals(Object o) {
  if(!(o instanceof MyType))
    return false;
  MyType mt = (MyType) o;
}
```

<br/>

## equals 올바른 구현 방법

- __구현단계__

```java
@Override public boolean equals(Object o) {
        if (o == this) // (1) == 연산자를 사용해 입력이 자기 참조인지 확인 => 자기 자신이면 true (성능최적화용도)
            return true;
        if (!(o instanceof PhoneNumber)) // (2) instanceof 연산자로 입력이 올바른 타입인지 확인. 그렇지 않으면 false 반환.
            return false;
        PhoneNumber pn = (PhoneNumber)o; // (3) 입력을 올바른 타입으로 형변환한다.
        return pn.lineNum == lineNum && pn.prefix == prefix // (4) 아래 설명참조
                && pn.areaCode == areaCode;
    }
```
  4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
      - 기본 타입 필드(float, double 제외)는 == 연산자로 비교
      - 참조 타입은 각각의 equals 메서드로 비교
      - float, double은 Float.compare, Double.compare로 비교
        - 부동소수값 때문이다. but 오토박싱으로 성능상 안좋음.
      - 배열의 모든 원소가 핵심필드면 Arrays.equals
      - null도 정상값으로 취급해야하면 Object.equals()로 비교
        - NullPointerException 방지
      - 아주 복잡한 필드를 가진 클래스는 필드의 표준형(canonical form)을 저장해둔 후 표준형끼리 비교하면 훨씬 경제적

<br/>

## 주의 사항

- __어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우한다.__
  - 비교하는 비용이 싼 필드 먼저 비교
  - 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.
    - ex) 동기화용 lock 필드
  - 핵심필드로부터 계산해낼 수 있는 파생 필드 역시 굳이 비교할 필요는 없지만, 객체 전체의 상태를 대표하는 파생필드쪽을 비교하는게 더 빠를때도있다.

<br/>

- TIP
  - AutoValue 프레임워크 사용하면 어노테이션으로 eqauls와 hashCode를 자동으로 만들어준다.

<br/>

## 용어 정리

#### 표준형? canonical form?

- 만약 equals할때마다 객체를 대문자로 바꾸고 연산하면 비용이드니깐, 미리 객체 만들때 '표준형'을 저장해두고 연산


```java
public final class CaseInsensitiveString {

  private final String s;
  private final String sForEquals; //equals를 단순화 하기 위한 필드

  public CaseInsensitiveString(String s) {
      if (s == null) {
          throw new IllegalArgumentException(); //NullPointerException()
      }
      this.s = s;
      this.sForEquals = s.toUpperCase(); //upper 또는 lower로 초기화
  }

  @Override
  public boolean equals(Object o) {
      return o instanceof CaseInsensitiveString &&
          ((CaseInsensitiveString) o).sForEquals.equals(this.sForEquals);
  }

  @Override
  public int hashCode(){
      return sForEquals.hashCode();
  }
  // remainder omitted
}
```
- 출처
  - https://github.com/2021BookChallenge/Effective-Java/issues/7
  - https://stackoverflow.com/questions/24427586/what-is-a-canonical-representation-of-a-field-meant-to-be-for-equals-method/24428722#24428722
