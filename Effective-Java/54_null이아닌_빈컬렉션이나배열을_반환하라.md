# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## null 반환 메서드
- 컬렉션, 배열이 비었을때 null 반환하는 몌서드는 사용하기 어렵고 오류 처리 코드도 늘어난다.

- **따라서 null이 아닌, 빈 컬렉션이나 배열을 반환하라**
```java
// 빈컬렉션을 반환하는 올바른 예
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

<br/>

## 빈 컬렉션 성능 문제?
#### 1) 성능 분석결과 성능 저하 주범이라고 확인되지 않는 한 이정도는 신경 안써도된다.
- 아이템 67

#### 2) 빈컬렉션과 배열은 굳이 새로 할당하지 않고 반환할 수 있다.
- 매번 같은 빈 불변 컬렉션을 반환하면된다.
  - 불변 객체는 자유롭게 공유해도 안전하다.
  - ex) `Collections.emptySet`, `Collections.emptyMap` 등
  - 최적화는 꼭 필요할때만 사용하자. 실제로 성능 개선되는지 확인.
```java
// 빈 컬렉션을 매번 새로 할당하지 않아도 된다.
public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```
```java
// 길이가 0일 수도 있는 배열을 반환하는 올바른 방법
public Cheese[] getCheeses(){
    return cheesesInstock.toArray(new Cheese[0]);
}
```
```java
// 빈 배열을 매번 새로 할당하지 않도록 했다. - 길이 0인 배열은 모두 불변
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
- toArray 메서드에 건낸 길이 0 배열은 원하는 반환 타입을 알려주는 역할
#### 안좋은예
```java
// 나쁜 예 = 배열을 미리 할당하면 성능이 나빠진다.
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```
- toArray메서드에 주어진 배열이 충분히 크면 해당 배열에 원소를 담아 반환하고, 그렇지 않으면 배열을 새로만들어 그 안에 원소를 담아 반환한다.
  

<br/>

## 결론
null이 아닌, 빈 배열이나 컬렉션을 반환하라.