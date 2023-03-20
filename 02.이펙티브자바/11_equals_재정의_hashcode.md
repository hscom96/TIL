# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라



- 핵심 : __equals를 재정의한 모든 클래스는 hashCode도 같이 재정의해야한다.__
  - 어기면 HashMap, HashSet 같은 컬렉션 원소로 사용할 때 문제를 일으킨다.

<br/>

## hashCode 일반 규약

- __equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 메서드는 일관되게 항상같은 값을 반환한다.__
- __equals(Object)가 두 객체를 같다고 판단하면, 두 객체의 hashCode는 같은 값을 반환한다.__
  - 즉, 논리적으로 같은 객체는 항상 같은 해시코드 반환
- __equals(Object)가 두 객체를 다르게 판단하더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.__
    - 단, 서로 다른 해시코드를 반환해야 해시테이블 성능 좋아짐
      - 최악의 경우(모든 hashCode가 같은 경우) 헤시테이블 성능이 O(n)이된다. (마치 링크드리스트)
      - 모두 다른 경우는 성능이 O(1)

<br/>

## 좋은 hashCode 작성법

> - 용어정리  
>   - 핵심 필드 : eqauls 비교에 사용되는 필드

1. int 변수 result 선언 후, 객체 첫번째 핵심필드를 아래 2-1단계 방식으로 계산한 해시코드로 초기화한다.
2. 객체의 핵심 필드 f 각각에 대해서 다음을 수행
    - 2-1) 해당 필드의 해시코드 c를 계산한다.
      - 기본 타입 필드라면, Type.hashCode(f) 수행 (Type은 해당 기본타입 박싱클래스)
      - 참조 타입 필드고 이 클래스의 equals메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 필드의 hashCode를 재귀적으로 호출한다.
        - 계산이 복잡해 질거 같으면, 필드의 표준형(ca-nonical representation)만든다.
        - 필드 값이 null이면 전통적으로 0 사용
      - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
        - 각 핵심 원소의 핵심 원소의 해시코드를 재귀적으로 계산
        - 배열에 핵심 원소가 없다면 상수(보통 0) 사용한다.
        - 모든 원소가 핵심 원소라면 Arrays.hashCode 사용
    - 2-2) 계산한 해시코드 c로 result 갱신한다.
      - ```result = 31 * result + c;```
> - 31로 정한 이유는 홀수이면서 소수이기 때문이다.
> - 짝수고 오버플로가 발생한다면 정보를 잃게된다. 2를 곱하는 것은 시프트 연산과 같은 결과를 내기 때문이다.   

3. result 반환



<br/>

## 실제 적용
### 방법 #1
- __적절한 방식__
```java
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

### 방법 #2
  - __간단한 대신, 성능이 느리다.__
    - 성능 민감하지 않은 상황에서만 사용하자.
    - 입력 인수를 담기 위한 배열이 만들어지고, 기본타입이 있다면 박싱과 언방식이 이루어지므로
```java
@Override public int hashCode() {
  return Object.hash(lineNum, prefix, areaCode);
}
```

### 방법 #3
  - 클래스가 불변이고 해시코드 계산비용 크다면, 매번 계산보단 캐싱하는 방식 고려하자.
    - __lazy initializiation(지연 초기화) 전략__
    - __인스턴스가 만들어질 때 해시코드를 계산해두는 전략__
```java
private int hashCode; // 자동 0 초기화
@Override public int hashCode() {
  int result = hashCode;
  if (result == 0){
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
  }
  return result;
}
```

<br/>

## 주의사항
  - 파생 필드는 해시코드 계산에서 제외해도 된다.
    - 즉, 다른 필드로부터 계산가능한 필드는 모두 무시해도 된다.
  - equals 비교에 사용되지 않은 필드는 반드시 제외해야한다.
  - 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.
    - 속도는 빨라져도, 헤시테이블 성능이 저하될 수 있다.
