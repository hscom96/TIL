# 아이템 86. Serializable을 구현할지는 신중히 결정하라

## `Serializable` 이란?

어떤 클래스의 인스턴스를 직렬화할 수 있게하려면 클래스 선언에 `implements Serializable`만 붙이면 된다.

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;
}
```

<br/>

## 첫 번째 문제 :  `Serializable`을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

- 직렬화 형태도 하나의 공개 API가 되므로, 그 직렬화 형태도 영원히 지원해야한다.
- 기본 직렬화 형태에서는 클래스의 priavte, package-private 인스턴스 필드마저 API로 공개가 되어 캡슐화가 깨진다.
    - (아이템15)의 필드로의 접근을 최대한 막아 정보를 은닉할 수 없다.
- 또 클래스 내부 구현을 수정하면 원래 직렬화 형태가 달라진다.
    - 한쪽은 구버전 인스턴스를 직렬화하고, 다른 쪽은 신버전 클래스로 역직렬화한다면 실패한다.

### 직렬화가 클래스 개선 방해하는 사례

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L; // serialVersionUID 꼭 명시 할 것 !
    
    private String name;
    private String email;
    private int age;
    private String address;
}
```

- 만약 필드 `serialVersionUID`에 번호를 명시 하지 않으면, 자동으로 고유 식별 번호를 부여한다.
- 이때 자동으로 생성된 고유 식별 번호는 클래스 이름, 맴버, 구현한 인터페이스 등으로 해시 함수를 적용한다.
- 그래서 이들 중 하나라도 수정한다면 UID 값도 변해서 쉽게 호환성이 깨진다.

띠라서 serialVersionUID 꼭 명시 하자

<br/>

## 두 번째 문제 : `Serializable`구현은 버그와 보안 구멍이 생길 위험이 높아진다.

- 객체는 생성자를 사용해 만드는게 기본이다. 하지만 직렬화는 언어의 기본 메커니즘을 우회하는 객체 생성 기법이다.
- 기본 방식을 따르든 재정의해 사용하든, 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'다.
- 이 생성자는 전면에 드러나지 않으므로 "생성자에서 구축한 불변식을 모두 보장해야하고 생성 도중 공격자가 객체 내부를 들여다 볼수 없도록 해야한다" 를 떠올리기는 쉽지 않다.

즉 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다는 뜻이다.

### 예시

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    private String address;
    private String email;

    public Member(String name, int age, String address) {
        if(age <= 19){
            throw new IllegalArgumentException();
        }
        this.name = name;
        this.age = age;
        this.address = address;
    }
}    
```

- 만약 Member가 생성될때 age가 19살 이하면 오류던지고 싶다.
- 하지만 직렬화를 통한 객체생성은 불변식을 무시하고 해당 객체에 19살 이하 나이를 가진채로 생성된다.

<br/>

## Serializable 구현 여부는 가볍게 결정할 사안이 아니다.

- 자바 직렬화 이용하는 프레임워크용 클래스라면 선택의 여지가 없다.
- 하지만 Serializable 구현에 따른 비용이 적지 않으니, 클래스를 설계 할때마다 그 이득과 비용을 잘 저울질해야 한다.

<br/>

## 세 번째 문제 : `Serializable` 구현은 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.

- 직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 반대도 가능한지 검사해야한다.

<br/>

## 상속용 클래스, 인터페이스의 경우

- 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안 되며, 인터페이스도 대부분 Serializable을 확장해서는 안 된다.
    - 지키지 않으면, 확장하거나 구현하는 이에게 커다란 부담을 준다.
- 상속용 클래스인데 직렬화를 지원하지 않으면 그 하위 클래스에서 직렬화를 지원하려할때 부담이 늘어난다.

<br/>

## 주의사항 (클래스가 직렬화, 확장 가능하면)

- 인스턴스 필드 값 중 불변식 보장해야할 게 있다면 하위 클래스에서 `finalize`메서드를 재정의하지 못하게 final로 선언하자.
- 인스턴스 필드 중 기본값으로 초기화되면 위배되는 불변식이 있다면 `readObjectNoData`메서드 추가하자.

<br/>

## 내부 클래스는 직렬화를 구현하면 안된다.

- 내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다.
- 이 필드들이 클래스 정의에 어떻게 추가되는지도 정의되지 않아서 내부 클래스에 대한 기본 직렬화 형태는 분명하지 않다.

## 참조

- 이펙티브 자바
- https://github.com/Meet-Coder-Study/book-effective-java/blob/main/12%EC%9E%A5/86_Serializable%EC%9D%84_%EA%B5%AC%ED%98%84%ED%95%A0%EC%A7%80%EB%8A%94_%EC%8B%A0%EC%A4%91%ED%9E%88_%EA%B2%B0%EC%A0%95%ED%95%98%EB%9D%BC_%EA%B9%80%EC%9E%AC%EC%A4%80.md