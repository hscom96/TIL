# 아이템 02. 생성자에 매개변수가 많다면 빌더를 고려하라.

**정적 팩터리와 생성자는 모두 선택적인 매개변수가 많을때 적절히 대응하기 어렵다.**

### 방법 1. 생성자

* 예를들어
    * 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수 2개까지 받는 생성자... 선택적 매개변수 모두 받는 생성자 등등 모두 생성해야한다.
* 단점
    * 작성하고, 읽기 어려움
    * 만약 사용자가 설정하길 원치 않는 매개변수까지 포함되어 있어도 어쩔수없이 값을 설정해줘야함.
    * 매개변수가 바껴도 컴파일러는 못잡아서 런타임 오류 발생 가능성
    * 등등

### 방법 2. 자바빈즈 패턴(JavaBeans pattern)

매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8); //(1)
cocaCola.setCalories(100);
cocaCola.setSodium(35);
```

* 장점
    * 앞선 생성자의 단점 해결. 인스턴스를 만들기 쉽고, 생성자 방식보단 읽기 쉬운 코드
* 단점
    * 객체 하나를 만들려면 메서드를 여러개 호출해야함
    * 객체가 완성되기 전까지는 일관성이 무너진 상태에 있다.
        * 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 여지가 있다.
            * 예를들어 (1)에서 객체 사용하는 경우
        * 반면 생성자패턴은 매개변수들이 유효한지 생성자에서만 확인하면 일관성 유지
    * getter, setter가 있어서 불변 클래스를 만들지 못함.
    * (쓰레드 간에 공유 가능한 상태가 있으니까) 스레드 안정성 얻으려면 추가적인 수고(locking)필요하다.

### 방법 3. 빌더 패턴 (Builder pattern)

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

* 특징
    * 인자 순서 상관없다.
    * 인자 많을수록 안전하게 객체 생성 가능
    * 빌더패턴은 유효성 확인 할수 있다.
        * (최대한 일찍 발견하려면) 각 빌더 생성자와 메서드에서 수행
        * build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식 검사.
    * (?) 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.
    * 상당히 유연하다.
        * 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있다.
* 단점
    * 빌더 생성 비용이 크지는 않지만 성능에 민감하면 문제가될 수도 있다.
    * 생성자 패턴보다는 코드가 장황해져서 매개변수가 4개 이상은 되어야 값어치함
        * but 시간이 지날수록 매개변수는 증가하는걸 고려하자.
* 예제코드 참조
    * https://github.com/greekZorba/java-design-patterns/tree/master/builder

### \[실습] Lombok을 이용하여 builder 구현

앞서 builder를 모두 구하면 힘드니 Lombok라이브러리의 @Builder어노테이션을 활용하자.

* builder패턴 적용 클래스

```java
@Entity
@Table(name = "refund")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Refund {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private long id;
  @Embedded
  private Account account;
  @Embedded
  private CreditCard creditCard;
  @OneToOne
  @JoinColumn(name = "order_id", nullable = false, updatable = false)
  private Order order;
  @Builder(builderClassName = "ByAccountBuilder", builderMethodName = "ByAccountBuilder") // 계좌 번호 기반 환불, Builder 이름을 부여해서 그에 따른 책임 부여, 그에 따른 필수 인자값 명확
  public Refund(Account account, Order order) {
    Assert.notNull(account, "account must not be null");
    Assert.notNull(order, "order must not be null");
    this.order = order;
    this.account = account;
  }
  @Builder(builderClassName = "ByCreditBuilder", builderMethodName = "ByCreditBuilder")  // 신용 카드 기반 환불, Builder 이름을 부여해서 그에 따른 책임 부여, 그에 따른 필수 인자값 명확
  public Refund(CreditCard creditCard, Order order) {
    Assert.notNull(creditCard, "creditCard must not be null");
    Assert.notNull(order, "order must not be null");
    this.order = order;
    this.creditCard = creditCard;
  }
}
```

* 테스트

```java
public class RefundTest {
    ...
    ...
  @Test
  public void ByAccountBuilder_test() {
    final Refund refund = Refund.ByAccountBuilder() // 빌더 이름으로 명확하게 그 의도를 드러 내고 있습니다.
        .account(account)
        .order(order)
        .build();
    assertThat(refund.getAccount()).isEqualTo(account);
    assertThat(refund.getOrder()).isEqualTo(order);
  }
  @Test
  public void ByCreditBuilder_test() {
    final Refund refund = Refund.ByCreditBuilder() // 빌더 이름으로 명확하게 그 의도를 드러 내고 있습니다.
        .creditCard(creditCard)
        .order(order)
        .build();
    assertThat(refund.getCreditCard()).isEqualTo(creditCard);
    assertThat(refund.getOrder()).isEqualTo(order);
  }
}
```

* **필수 매개변수가 없을 경우 예외처리를 설정**할 수 있다.
* **builder 이름으로 책임을 부여하자.**
    * 예를들어, 주문에 대한 환불이 있을경우 신용카드 취소, 계좌 기반 환불이 있다.
    * 각각 받아야할 정보들이 다른데 하나의 builder로 검증하기 힘들다.
    * 이런경우 builder의 이름을 명확하게 해서 책임을 부여하자.

***

## 참조

* **Lombok 사용법**
    * https://github.com/cheese10yun/blog-sample/tree/master/lombok
* **Lombok에서 builder 안전하게 사용법**
    * https://www.popit.kr/builder-%EA%B8%B0%EB%B0%98%EC%9C%BC%EB%A1%9C-%EA%B0%9D%EC%B2%B4%EB%A5%BC-%EC%95%88%EC%A0%84%ED%95%98%EA%B2%8C-%EC%83%9D%EC%84%B1%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/
* Lombok builder 예제
    * https://zorba91.tistory.com/298
