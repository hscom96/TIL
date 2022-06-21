# 아이템 13. clone 재정의는 주의해서 진행하라

## 시작하기 앞서 ..
### 복제는 왜 하는거야?
- __객체를 복제하는 이유는 원본 객체를 안전하게 보호__ 하기 위해서다.
  - 신뢰하지 않는 영역으로 원본 객체를 넘겨 작업할 경우 원본 객체가 훼손될 수 있어서 복제본을 만든다.

  (출처 : 이것이 자바다)
  <br/>

### 얕은 복제?

- __얕은 복제는 단순히 필드값을 복사해서 객체를 복제한다.__
  - 기본 타입일 경우 값 복사가 일어나고,
  - __참조 타입일 경우 객체 번지가 복사된다.__
    - 따라서 복제 객체에서 참조객체를 변경하면 동시에 변경된다.

<br/>

- __Object의 clone() 메소드는 얕은 복제된 객체를 리턴한다.__

<br/>

### 깊은 복제?

__- 깊은 복제는 참조하고 있는 객체도 복제한다.__

<br/>

- Objects의 Clone() 메소드를 재정의 해야한다.

<br/>

![copy](https://jaehun2841.github.io/2019/01/13/java-object-copy/clone.jpg)

<br/>

## Cloneable 특이함

> - __인터페이스를 상당히 이례적으로 사용__
>   - 인터페이스를 구현한다는 것은 보통 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 것
>   - Cloneable 경우에는 상위 클래스에 정의된 메서드의 동작 방식을 변경한 것

- __Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스__

<br/>

- __clone 메서드는 Cloneable이 아닌 Object에 protected 메서드로 선언되어 있다.__

<br/>

- __Cloneable은 Object의 clone의 동작 방식을 결정한다.__
  - Closeable을 구현할 클래스에서 clone 호출하면, 필드를 복사한 객체 반환
  - 구현 안한 클래스에서 clone 호출하면, 예외 발생

<br/>

## Cloneable 일반 규약  
#### 1) x.clone() != x는 참이다.  
  - 원본 객체와 반환 객체는 서로 다른 객체다.
#### 2) x.clone().getClass() == x.getClass()는 참이지만 반드시 만족해야 하는 것은 아니다.  
#### 3) x.clone().equals(x)는 참이지만 필수는 아니다.  
#### 4) x.clone().getClass() == x.getClass()  
  - 관례상 clone메서드는 super.clone을 호출해서 반환 객체를 얻어야한다.
    - 모든 상위 클래스가 관례를 따르면 참이다.
  - 관례상 반환된 객체와 원본 객체는 독립적이여야함.
    - super.clone으로 얻은 객체의 필드 중 하나 이상을 수정해야할 수 있다.

<br/>

> 만약, clone()에서 super.clone이 아닌 new 생성자를 호출하게되면 컴파일러에는 문제가 안된다.  
> 하지만 하위 클래스에서 상위 타입이 반환되어 오류 발생

<br/>

### Cloneable 주의할점
- __단순히 super.clone 결과를 그대로 반환하면? 참조필드는 같은 객체를 참조하게 된다.__
  - 원본, 복제본 중 하나를 수정하면 다른 한 곳도 수정된다.
  - __clone 메서드는 원본 객체에 해를 끼치지 않고, 복제된 객체에 불변식을 보장해줘야한다.__

<br/>

- __가변객체 필드가 final이면 동작하지 않는다.__
  - final은 새로운 값 할당 못하기때문 (제거해야 될수도..)
  - __Cloneable 아키텍처는 '가변객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.__

<br/>

- __재정의될 수 있는 메서드를 호출하지 않아야한다.__
  - (이해안감)원본과 복제본의 상태가 달라질 가능성

<br/>

- (이해안감)상속용 클래스는 Cloneable을 구현해서는 안된다. (아이템19 두가지 모두)

<br/>

## Cloneable 적용

<br/>

### 1) 불변 객체경우 (가변 상태 참조 x)
-  __불필요한 객체 생성을 지양하는게 좋으므로(아이템6) 불변 클래스는 굳이 clone 메서드를 제공하지 말자.__

<br/>

```java
public final class PhoneNumber implements Cloneable {
    @Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  
        }
    }
    // 나머지 코드 생략
}
```
<br/>

### 2) 객체가 가변 객체를 참조하는 경우

#### 해결법 #1
- super.clone 호출한 후 필요한 필드 전부 적절히 수정
  - 가변 객체를 복사하고 복제본을 가리키게한다.
- 배열은 clone() 호출가능
```java
public class Stack implements Cloneable {
    private Object[] elements;
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone(); // 배열(elements)은 clone()을 호출하여 해결가능하다.
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    // 나머지 코드 생략
}
```

- 해결법 #2
  - 재귀적으로 연결리스트 복사하는 방법
  - 리스트 길면 오버플로 일으킬 위험
- 해결법 #3
  - 반복자를 써서 연결리스트 복사하는 방법
- 해결법 #4
  1. super.clone을 통해 초기상태 설정
  2. 원본 객체 상태를 다시 생성하는 고수준 메서드 호출

<br/>

## 복사 생성자와 복사 팩터리 (추천방법)
>  Cloneable을 이미 구현한 클래스를 확장하는건 어쩔 수 없다.

- __기본으로 이 방식을 사용해야한다.__


<br/>

#### 복사 생성자
`public Yum(Yum yum) { ... };`


#### 복사 팩터리
- 정적 팩터리(아이템1)와 같다.

`public static Yum newInstance(Yum yum) { ... };`

### 장점
- 위험한 객체 생성 메커니즘(생성자 사용 x)를 사용하지 않는다.
- 정상적인 final용법과도 충돌나지 않는다.
- 불필요한 검사 예외를 던지지 않는다.
- 형변환 불필요
- 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
  - 클라이언트는 원본의 구현 타입에 얽매이지 않고, 복제본의 타입을 직접  선택 가능하다.
  - ex) HashSet 객체 s를 TreeSet 타입으로 복제 가능하다.




<br/>

## 용어 정리
- 비검사 예외? 검사 예외?
