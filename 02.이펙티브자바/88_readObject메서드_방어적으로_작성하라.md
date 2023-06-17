# 아이템 88. readObject 메서드는 방어적으로 작성하라

## `readObject` 메서드

- `readObject`는 바이트 스트림을 받는 생성자라고 할 수 있다.
    - `readObject` 메서드는 실질적으로 또 다른 생성자이기 때문에 같은 수준으로 불변식을 보장하기 위해서 주의를 기울여야한다.
    - 생성자처럼 인수가 유효한지 검사하고, 필요하면 매개변수를 방어적으로 복사해야한다.

<br/>

## 유효성 검사

```java
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {

    // 불변식을 만족하는지 검사한다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "after" + end);
    }
}
```

<br/>

## 방어적 복사

- 정상적인 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변적인 Period 인스턴스를 만들어낼 수 있다.
- 공격자가 역직렬화를 통해 바이트 스트림 끝에 추가된 악의적인 객체 참조를 읽으면 Period의 내부 정보를 얻을 수 있다.
- 이 참조를 이용하여 인스턴스를 수정할 수 있다. 즉, 불변이 아니게 되는 것이다.
- 역직렬화를 할 때는 클라이언트가 접근해서는 안 되는 객체 참조를 갖는 필드는 모두 방어적으로 복사를 해야 한다.

```java
// 방어적 복사와 유효성 검사를 모두 수행한다.
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if (start.compareto(end) > 0) {
        throw new InvalidObjectException(start + " after " + end);
    }
}
```

> - 한편 final 필드는 방어적 복사가 불가능하다. 그래서 readObject 메서드를 사용하려면 final 제거하자. 공격 위험에 노출되는것 보다 낫다.

<br/>

## 그럼 언제 기본 readObject를 사용할까?

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사없이 필드에 대입하는 public 생성자를 추가하면 안된다고 판단되면?
    - 안된다면 커스텀 `readObject`메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야한다.
    - 가장 추천하는건 직렬화 프록시 패턴을 사용하는 방법이다. (안전하게 만드는데 필요한 노력을 줄여줌)
- final이 아닌 직렬화 가능한 클래스라면 생성자처럼 readObject 메서드도 재정의(overriding) 가능한 메서드를 호출해서는 안 된다. (아이템 19)
    - 하위 클래스의 상태가 완전히 역직렬화 되기 전에 하위 클래스에서 재정의된 메서드가 실행되기 때문이다.

<br/>

## 안전한 readObject 메서드 작성 지침

- private 이여야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라.
- 모든 불변식을 검사하고, 어긋난다면 InvalidObjectException을 던져라.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation를 사용하라.
- 재정의(Overriding) 가능한 메서드는 호출하지 말자.