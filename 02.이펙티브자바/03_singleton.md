아이템 03. private 생성자나 열거 타입으로 싱글턴임을 보증하라
===========================================================

## 싱글턴이란?

인스턴스를 오직 하나만 생성할 수 있는 클래스를 뜻한다.

싱글턴을 만드는 방식은 두 가지가 있다.  
**두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 사용한다.**

<br/>

## 첫 번째 방법

**public static 멤버가 final인 방식이다.**

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
    }
}
```

public, protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 단 하나뿐임이 보장된다.

- 장점

	-	static 팩터리 메소드를 사용하는 방식보다 명확하고 간단하다.

- 단, 리플렉션 api를 호출해 private 생성자를 호출할 수 있다.

	-	공격을 방어하려면 생성자를 수정하여 두번째 객체가 호출될때 예외발생시켜야한다.

<br/>

## 두 번째 방법

**정적 팩터리 메서드를 public static 멤버로 제공한다.**

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {
    }
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 또 다른 인스턴스가 생기지 않는다.

- 장점 - api를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
    - 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 구성할 수도 있다. - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
    - (이해안감) static 팩토리 메소드를 Supplier<Elvis>에 대한 메소드 레퍼런스로 사용할 수도 있다.

<br/>

## 직렬화

두 방식 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다.

문제를 해결하려면  
모든 인스턴스 필드에 transient를 추가 (직렬화 하지 않겠다는 뜻) 하고 readResolve 메소드를 다음과 같이 구현하면 된다.

```java
private Object readResolve() {
  // '진짜' Elivis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
         return INSTANCE;
    }
```

<br/>

## 세 번째 방법

원소가 하나인 열거타입을 선언한다.

```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding(){...}
}
```

- 장점 - 조금 더 간결하다. - 추가 노력없이 직렬화할 수 있다. - 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일 막아준다.

**대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

- 단 만들려는 싱글턴이 enum 외 클래스를 상속해야한다면 사용할 수 없다.

<br/>

## 참조

- 직렬화 : readResolve와 writeReplace

	-	https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method

- 싱글턴 패턴 생성방법 및 주의사항

	-	https://webdevtechblog.com/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36
