# 아이템 36. 비트 필드 대신 EnumSet을 사용하라

## 비트필드 열거 상수
```java
public class Text{
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  ...
}
```
- 비트연산을 통해 집합연산 (합집합, 교집합) 효율적으로 수행가능
- 그래서 상수집합을 주고 받을때 사용
### 단점
#### 정수 열거 상수의 단점 그대로 지닌다.
#### 비트필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 떄보다 해석하기가 더 어렵다.
#### 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입 선택해야한다.

<br/>

## (대안) EnumSet
- EnumSet 클래스는 __열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현__ 해준다.
```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```
- EnumSet 클래스는 Set인터페이스를 완벽히 구현 & 타입 안전
- EnumSet 내부는 비트 벡터로 구현되어있으며, 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트필드에 비견되는 성능을 보여준다
- removeAll과 retainAll 과 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현


<br/>

## 결론
- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 __비트 필드를 사용하지 말자.__
- 그대신 EnumSet!
