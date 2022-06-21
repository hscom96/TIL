# 아이템 14. Comparable을 구현할지 고려하라



 > Comparable 인터페이스의 유일한 메서드인 compareTo는 Object 메서드가 아니다.

 ## Object의 equals와 다른점
 - compareTo는 equals와 다르게
    1. 단순 동치성 비교를 더해, __순서까지 비교가능__
    2. 제네릭하다.

<br/>

- __알파벳, 숫자, 연대 같이 순서를 고려해야 하는 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.__
  - ex1) `Arrays.sort(a)`같이 손쉽게 정렬가능
  - ex2) TreeSet, TreeMap, Collections, Arrays 등 정렬,검색,비교 컬렉션에 유용 (컬렉션은 equals가 아닌 compareTo 사용)


<br/>

## compareTo 일반규약

> - 객체의 순서를 비교한다.
>   - 이 __객체가 주어진 객체보다 작으면 음수, 같으면 0, 크면 양수 반환__, 비교할 수 없는 타입은 ClassCastException

> - sgn()은 부호함수 (음수:-1, 0, 양수:1 반환)

- 아래 3가지 규약은 equals 규약과 같이 반사성, 대칭성, 추이성을 충족해야 한다는 뜻이다.

#### 1. 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
- 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다는 이야기

#### 2. 추이성 보장해야한다.
- 첫번째가 두번째보다 크고 두번째가 세번째보다 크면 첫번째는 세번째보다 커야한다는 이야기

#### 3. 모든 z에 대해서 x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))
- 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다는 이야기

#### 4. (x.compareTo == 0)== (x.equals(y))
  - 필수는 아니지만 꼭 지키는게 좋음

<br/>

## 주의사항
> equals 주의 사항과 비슷하다.

- 기존 클래스에서 확장한 구체클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 수 없다. (아이템 10 참조)
  - 해결법은?
    1. 확장하는 대신 독립된 클래스를 만들고
    2. 이 클래스에 원래 클래스의 인스턴스를 가리키는 필드를 두자.
    3. 내부 인스턴스를 반환하는 view 메서드를 제공

<br/>

## compareTo 작성요령
#### 1) CompareTo는 제네릭 인터페이스다. 입력 인수의 타입을 확인하거나 형변환할 필요없다.
  - 타입 잘못되면 컴파일이 안된다.

- null이 인수면 NullPointerException 던져야함.

<br/>

#### 2) compareTo 메서드는 각 필드가 동치인지를 비교하는게 아니라 순서를 비교한다.
  - 참조필드 비교는 compareTo 재귀적으로 호출
  - Comparable 구현하지 않은 필드나 표준이 아닌 순서로 비교해야한다면, Comparator를 사용한다.

<br/>

#### 3) 정수 기본 타임 필드를 비교할때 관계연산자(>, <)를 사용하는 비교 방식은 추천하지 않는다.
- 거추장, 오류 유발 가능성이 있다.
- 아래 방식 추천
  - __박싱 기본 타입 클래스들에 새로 추가된 정적 메서드 compare 사용__
  - 실수 기본타입필드 비교할땐 Double.compare, Float.compare

<br/>

#### 4) 클래스에 여러 필드가 있으면 핵심 필드부터 검사하자.

```java
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode); // 첫번째중요
        if (result == 0)  {
            result = Short.compare(prefix, pn.prefix); // 두번째중요
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum); // 세번째중요
        }
        return result;
    }
```

<br/>

#### 5) 비교자 생성 메서드(comparator construction)를 활용하는 방법
- 간결하지만, 약간의 성능저하가 있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
           comparingInt((PhoneNumber pn) -> pn.areaCode)
                   .thenComparingInt(pn -> pn.prefix)
                   .thenComparingInt(pn -> pn.lineNum);

   public int compareTo(PhoneNumber pn) {
       return COMPARATOR.compare(this, pn);
   }
```
