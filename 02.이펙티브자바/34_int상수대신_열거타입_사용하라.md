# 아이템 34. int 상수대신 열거 타입을 사용하라

## 열거 타입이란?

- 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외값은 허용하지 않는 타입

<br/>

## 정수 열거 패턴

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

### 단점

#### 타입 안전하지 않고, 표현력 좋지 않다.

- 동등연산자 비교해도 아무런 경고 메시지 출력하지 않는다. => 타입 안전 x
    - `APPLE_FUJI == ORANGE_NAVEL;`
- 별도 namespace를 지원하지 않아서 어쩔 수 없이 접두어를 써서 이름 충돌 방지한다.
    - `APPLE_` , `ORANGE_`

#### 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

- 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.
- 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.

#### 정수 상수는 문자열로 출력하기가 다소 까다롭다.

```java
public static final int APPLE_PIPPIN = 1;
System.out.println(APPLE_PIPPIN); // 문자 APPLE_PIPPIN가 아닌 1이 출력된다.
```

<br/>

## 문자열 열거 패턴

```java
// 내가 생각한 예시
public static final String APPLE = "APPLE";
public static final String GRAPE = "GRAPE";
public static final String ORANGE = "ORANGE";
```

### 단점

- 정수 열거 패턴보다 더 나쁨
- 필드명 대신 string 상수 값을 클라이언트 코드에 하드 코딩 => 문자열에 오타가 있어도 컴파일러는 확인 못함 => 런타임 버그
- 문자열 비교에 따른 성능저하

####

<br/>

## 열거 타입

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
```

### 장점

#### 열거 타입은 실제로는 클래스다.

- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개, 생성자 제공 x
- 따라서 열거 타입으로 만들어진 인스턴스들은 딱 하나씩만 존재
- 싱글턴은 원소가 하나뿐인 열거 타입, 거꾸로 열거 타입은 싱글턴을 일반화한 형태

#### 열거 타입은 컴파일타임 타입 안정성 제공

- 열거타입을 매개변수로 받는 메서드를 선언하면 다른 타입 값을 넘기려고 하면 컴파일 오류

#### 열거 타입에는 각자의 namespace가 있어서 이름이 같은 상수도 평화롭게 공존한다.

- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 안해도 된다.
- 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다. (공개되는 것은 오직 필드 이름뿐)

#### 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.

#### 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스 구현 가능

#### Object메서드, Comparable, Serializable들을 잘 구현해놓음

<br/>

## 열거타입 예시

- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장
- 불변이므로 모든 필드는 final

```java
enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
    // ...

    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면중력

    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```

- 열거 타입에만 유용한 메서드는 클라이언트에 노출할 이유가 없으면 private, package-private으로 선언
- 널리 쓰이는 열거타입은 톱레벨 클래스로, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다. (아이템24)

<br/>

## 상수별 메소드 구현

#### 개선 전

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```

- 깨기 쉬운 코드다.
    - 새로운 상수를 추가하면 해당 case 문도 추가해야한다.
    - 깜빡하고 추가안하고 연산을 수행하려 하면 런타임 오류 발생

#### 개선 후 (상수별 메소드 구현)

```java
enum Operation {
    PLUS {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        public double apply(double x, double y) {
            return x - y;
        }
    }
    // ...
    public abstract double apply(double x, double y);
}
```

- apply가 추상 메서드이므로 재정의하지 않으면 컴파일 오류

#### 개선 후 (상수별 메소드 + 데이터 구현)

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

```java
public static void main(String []args){
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
       System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

#### 열거 타입용 fromString 메서드 구현하기

```java
private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

// fromString은 toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 기능
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

- Operation 상수가 stringToEnum 맵에 추가되는 시점은 __열거 타입 상수 생성 후 정적 필드가 초기화__ 될 때다.
- 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다.
    - 열거 타입 생성자가 실행되는 시점에는 정적 필드 들이 아직 초기화 되기 전이라, __열거 타입 생성자에서 정적 필드를 참조하려고 하면 컴파일 에러__  발생한다.
- 열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.

<br/>

## 전략 열거 타입 패턴

#### 개선전 문제점

- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 다음예 참조.
    - 열거 타입 추가시 case문 추가를 잊어 오작동할 가능성 존재

```java
public enum PayrollDay {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        //기본 급여
        int basePay = minutesWorked * payRate;
		//잔업수당
        int overtimePay;
        switch (this) {
        	//주말
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            //주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
```

- 아래 두 방식 모두 가독성 크게 떨어지고, 오류 발생 가능성 높아짐
    1. 잔업수당 계산 코드를 모든 상수에 중복해서 넣기
    2. 계산코드를 평일용, 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 필요한 메서드를 호출
- 아래 방식도 switch문 쓸때와 같은 단점. 새로운 상수 추가하면서 메서드 재정의 안하면 오작동.
    3. 평일 잔업수당 계산용 메서드만 구현해놓고, 주말 상수에서만 재정의

#### 개선후 (전략 열거 타입 패턴)

- 새로운 상수를 추가할 때 잔업수당 전략을 선택
- 잔업수당 계산을 private 중첩열거타입(`PayType`)으로 옮기고
- `PayrollDay`열거 타입의 생성자에서 이 중 적당한 것을 선택한다.

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

#### switch문이 좋을 경우는?

1. 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

```java
// 각 연산의 반대 연산을 반환
public static Operation inverse(Operation operation) {
        switch (operation) {
            case PLUS:
                return Operation.MINUS;
            case MINUS:
                return Operation.PLUS;
            case TIMES:
                return Operation.DIVDE;
            case DIVDE:
                return Operation.TIMES;
        }
        throw new AssertionError("알 수 없는 연산 : " +operation);
}
```

2. 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이여도 이 방식 적용.

<br/>

## 열거 타입을 언제 쓰는가?

- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거타입을 사용하자.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.
