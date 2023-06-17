# 아이템 52. 오버로딩는 신중히 사용하라

## 오버로딩

- **오버로딩된 메서드는 어느 메서드를 호출할지 컴파일타임에 정해진**다.

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c)); // 예상과다르게 반환 결과는 "그 외"만 3번 연달아 출력한다.
    }
}
```

- 컴파일 타임에는?
    - for문의 c는 항상 `Collection<?>` 타입
    - 따라서 `classify(Collection<?>)`만 호출한다.
- 런타임에는?
    - 타입이 매번 달라지지만, 호출할 메서드를 선택하는데는 영향 주지 못한다.

#### 해결책은?

- 모든 `classify`를 하나로 합친 후 `instanceof`으로 명시적으로 검사

```java
 public static String classify(Collection<?> c) {
        return c instanceof Set  ? "집합" :
                c instanceof List ? "리스트" : "그 외";
}
```

<br/>

## overriding(재정의)

- **overriding(재정의)한 메서드는 해당 객체의 런타임 타입기준으로 메서드 호출**이 이루어진다.

```java
class Wine {
    String name() { return "포도주"; }
}
class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}
class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name()); // "포도주" "발포성 포도주" "삼페인" 출력
    }
}
```

<br>

## 주의사항

#### 매개변수 수가 같은 오버로딩은 웬만하면 만들지 말자.

#### 가변 인수를 사용하는 메서드는 오버로딩를 사용하지 말자.

#### 오버로딩대신 메서드 이름을 다르게 지어줄 수 있다.

- ex) `writeBoolean(boolean)`, `writeInt(int)` 등..
- 생성자의 경우 이름을 다르게 지을 수 없으므로 무조건 오버로딩이 된다.
    - 정적팩터리라는 대안을 활용해보자

#### 매개변수 개수가 같은 오버로딩을 피할 수 없다면?

- 매개변수 수가 같더라도 **서로 형변환할 수 없으면 헷갈릴 일 없다.**
- 하지만 서로 형변환할 수 없어도 항상 안전하진 않다
    - ex) `remove(int index)` 와 `remove(Object o)`
    - 제네릭, 오토박싱이 등장하면서 int를 Integer로 자동 변환해주니 근본적으로 달라지지 않게 되었다.

 ```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
list.remove(3); // index 3을 없애는지? 3을 지우는건지?
 ```

#### 메서드를 다중정의할때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.

- 서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않다.

<br/>

## 결론

- 되도록 매개변수 수가 같을 떄는 다중정의를 피하는 게 좋다.
- 굳이 해야한다면 헷갈릴만한 매개변수는 형변환하여 명확히 선택되도록한다.