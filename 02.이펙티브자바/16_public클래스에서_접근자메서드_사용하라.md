# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

<br/>

- __패키지 바깥에서 접근할 수 있는 클래스(public)라면 접근자를 제공함으로써__ 클래스 내부 표현 방식을 언제든 바꿀수 있는 유연성을 얻을 수 있다.
    - public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨나 내부 표현 방식을 마음대로 못 바꿈

<br/>

- 하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제없다.
    - 어차피 클라이언트도 패키지 안에서만 동작하므로 바깥 패키지 손 안대고 수정 가능

<br/>

- public 클래스의 필드가 불변이여도 직접 노출하면 단점은 덜해도 좋지 않다.
    - api를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.
