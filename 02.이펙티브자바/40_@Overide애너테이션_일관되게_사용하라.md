# 아이템 40. `@Overide`애너테이션을 일관되게 사용하라

## `@Overide` 애너테이션이란

- `@Overide`애너테이션은 메서드 선언에 달 수 있음
- 상위 타입의 메서드를 재정의 했음 뜻한다.

</br>

## 예시 (필요한 이유)

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size()); // 예상 : 26, 실제출력 : 260
    }
}
```

- 문제는 equals 메서드가 재정의(overriding)된게 아니라 다중정의(overloading)이 되었던 것이다.
    - 따라서 `Set`에서 의도치 않는 동작 발생

<br/>

## 결론

#### 상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override`애너테이션을 달자

- 일관되게 사용한다면 실수로 재정의했을때 컴파일러가 바로 알려줄 것이다.

> 예외로 상위 클래스의 추상메서드를 구현하는 구체 클래스에서는 어차피 구현안하면 컴파일러가 알려주니 굳이 안해줘도 된다.
