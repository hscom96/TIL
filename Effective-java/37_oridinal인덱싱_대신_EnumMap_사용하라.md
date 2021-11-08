# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
}
```

## (따라하지말자) ordinal() 배열 인덱스 사용 #1

```java
// 생애주기(한해, 여러해, 두해살이)별로 묶는 기능
public static void usingOrdinalArray(List<Plant> garden) {
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
        for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant plant : garden) {
            plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
        }

        for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
            System.out.printf("%s : %s%n",
                    LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
}
```

### 문제점

#### 1. 배열은 제네릭과 호환되지 않음 => 비검사 형변환

#### 2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야함

#### 3. 정수는 열거 타입과 달리 타입안전하지 않다.

- 정확한 정숫값을 사용한다는 것을 보증해야함

<br/>

## (개선) EnumMap

```java
public static void usingEnumMap(List<Plant> garden) {
    // EnumMap의 생성자가 받는 Class 객체는 한정적 타입토큰으로, 런타입 제네릭 타입 정보를 제공(아이템 33)
      Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

      for (LifeCycle lifeCycle : LifeCycle.values()) {
          plantsByLifeCycle.put(lifeCycle,new HashSet<>());
      }

      for (Plant plant : garden) {
          plantsByLifeCycle.get(plant.lifeCycle).add(plant);
      }

      System.out.println(plantsByLifeCycle);
}
```

### 장점

- 이전 ordinal 예제와 다르게 안전하지 않는 형변환 사용 x
- 출력용 문자열 제공 (toString())
- 타입안정성 O
- 배열 인덱스를 계산하는 과정에서 오류날 가능성 없음
- 짧고, 성능도 비등

<br/>

## 스트림

### EnumMap 사용 x

```java
//
System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle)));
```

#### 단점

- EnumMap 사용 안해서 공간, 성능 이점 사라짐

### EnumMap 사용

```java
System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())))
```

- 앞서 EnumMap만 사용한 예제와 다르게 해당 식물이 있을때만 중첩 맵을 만든다.

<br/>

## (따라하지말자) ordinal() 배열 인덱스 사용 #2

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 사용
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        // 한 상태에서 다른 상태로의 전이를 반환
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

### 단점

- Phase, Phase.Transition 열거 타입을 수정하면서 TRANSITIONS를 함께 수정하지 않거나 잘못 수정하면 런타임 오류
- TRANSITIONS의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지고 null로 채워짐

<br/>

## EnumMap

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap ㅋ= Stream.of(values())
                    .collect(Collectors.groupingBy(t -> t.from, // 바깥 Map의 Key
                            () -> new EnumMap<>(Phase.class), // 바깥 Map의 구현체
                            Collectors.toMap(t -> t.to, // 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                    t -> t, // 안쪽 Map의 Value
                                    (x,y) -> y, // 선언만 하고 실제로 안쓰임
                                    () -> new EnumMap<>(Phase.class)))); // 안쪽 Map의 구현체
        }

        public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }
}
```

- 여기에 새로운 상태를 추가해도 기존 로직에서 잘 처리해준다. => 유지보수성 좋다.
- 낭비되는 공간, 시간 거의 없다.
