# 아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라


## `<? extends E>` (한정적 와일드카드타입)

#### 사용 X (오류발생)

```java
// Stack<E> 객체 일부
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
numberStack.pushAll(integers); // 오류 발생 : 불공변 (Iterable<Number> vs Iterable<Integer>)
```

- (`Iterable<Number>` vs `Iterable<Integer>`) 매개변수화 타입이 불공변이기때문에 오류 발생한다.

#### 사용 O

> - E는 생산자 매개변수 : 즉 입력 매개변수로 부터 이 컬렉션으로 원소를 옮겨 담는다는 뜻

```java
// Stack<E> 객체 일부
public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
}
```

- 불공변에서 생기는 문제를 한정적 와일드카드타입으로 유연하게 대처가능
- `Iterable<? extends E>`는 E의 하위 타입의 Iterable이라는 뜻

<br/>

## `<? super E>` (한정적 와일드카드타입)

#### 사용 X (오류발생)

```java
// Stack<E> 객체 일부
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = new ArrayList<>();
numberStack.popAll(objects); // 오류 발생 : 불공변 (Collection<Object> vs Collection<Number>)
```

- `Collection<Object>`은 `Collection<Number>`의 하위 타입이 아니므로 오류발생

#### 사용 O

> E는 소비자 매개변수 : 즉 이 커렉션 인스턴스의 원소를 입력 매개변수로 옮겨 담는다는 뜻

```java
// Stack<E> 객체 일부
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

- `Collection<? super E>`는 E의 상위 타입의 Collection이라는 뜻

<br/>

## 주의사항

#### 유연성을 극대화하려면?

- 원소의 생산자나 소비자용 입력 매개변수에 __와일드카드 타입 사용__
    - ex) `public Chooser(Collection<? extends T> choices)` (아이템28)

#### 반면, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면?

- 타입을 정확히 지정해야하는 상황으로 __와일드카드 타입을 쓰지 말아야한다.__

#### 와일드 카드 사용 구분

- __PECS : producer-extends, consumer-super__
- 생산자는 `<? extends T>`, 소비자는 '<? super T>'

#### 반환 타입에는 한정적 와일드카드 타입을 쓰면 안된다.

- 그렇지 않으면 클라이언트 코드에서도 와일드카드를 써야됨
- `public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)` (아이템 30)

#### 클래스 사용자가 와일드 타입을 쓰였는지 모르게 신경 안쓰게하자.

#### Comparable은 언제나 소비자이므로 일반적으로 `Comparable<E>`보다는 `Comparable<? super E>`를 사용하는게 좋다.

- ex) (아이템30)`public static <E extends Comparable<? super E>> E max(List<? extends E> list)`
- Comparator도 마찬가지다!

<br/>

## 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.

- 규칙
    - 비한정적 타입 매개변수 -> 비한정적 와일드 카드
    - 한정적 타입 매개변수 -> 한정적 와일드 카드

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 후자는 더 간단한 메서드 signiture를 가지고 있어서 더 이해하기 쉬워서 공개 API에 좋다.
- 참고 : https://stackoverflow.com/questions/18142009/type-parameter-vs-unbounded-wildcard
    - 취향 차이라고함
    - 즉, 구현 용이함(제네릭 구현 더 쉬움) vs 서명 복잡성 감소(와일드카드버전은 형식 매개변수가 더 적음)

#### private 도우미 메서드

```java
public static void swap(List<?> list, int i, int j) {
      swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
      list.set(i, list.set(j, list.get(i)));
}
```

- List<?>에는 null이외 어떤 값도 넣을 수 없으므로 로타입, 형변환 안쓰고도 private 도우미 메서드를 활용 할 수있다.

<br/>

## Reference

- https://stackoverflow.com/questions/18142009/type-parameter-vs-unbounded-wildcard
