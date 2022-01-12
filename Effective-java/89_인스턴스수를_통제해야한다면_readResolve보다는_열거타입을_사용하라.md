# 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve 보단는 열거 타입을 사용하라

## 싱글턴일까?
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    ...
}
```
- Serializable을 구현하는 순간 더이상 싱글턴이 아니게 된다.
- 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스 반환하게 된다.

<br/>

## `readResolve` 메서드
- `readResolve`의 기능을 사용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다.
  - 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환한다.
  - 이때 새로 생성된 객체 참조는 가비지 컬렉션의 대상이 된다.
```java
// 개선여지있음
private Object readResolve() {
    // 기존에 생성된 인스턴스를 반환한다.
    return INSTANCE;
}
```
- 한편 여기서 살펴본 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 필요가 없으니 모든 인스턴스 필드는 transient 로 선언해야 한다.   
그러니까 readResolve 메서드를 인스턴스의 통제 목적으로 이용한다면 **모든 필드는 transient로 선언해야 한다.**
- 만일 그렇지 않으면 역직렬화 과정에서 역직렬화된 인스턴스의 참조를 가져올 수 있다. 즉, 싱글턴이 깨지게 된다. 

<br/>

## 해결책은 열거 타입
- 필드를 transient로 선언하여 해결할 수도 있지만, 원소 하나짜리 열거 타입으로 바꾸는편이 더 낫다.
- 열거 타입을 이용해 구현하면 선언한 상수 외 다른 객체는 존재하지 않음을 자바가 보장해준다.
  
```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

## 참조
- 이펙티브 자바 3판
- (싱글턴 깨지는 예제 해설**) https://github.com/Java-Bom/ReadingRecord/issues/164
- (싱글턴 깨지는 예제 해설**) https://stackoverflow.com/questions/37660696/elvisstealer-from-effective-java