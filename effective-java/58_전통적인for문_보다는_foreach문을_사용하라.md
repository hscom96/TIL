# 아이템 58. 전통적인 for 문보다는 for-each 문을 사용하라

## 전통적 for문

```java
// 오류 발생 (i.next() 내부에서 잘못호출) 
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(i.next(), j.next()));
```

#### 문제점

- 인덱스 변수는 코드를 지저분하게한다.
- 오류 생길 가능성 높다. (위 예시처럼)
- 컬렉션, 배열이냐에 따라 코드형태 달라짐

<br/>

## for-each문

```java
// 개선ver - 위 전통적 for문 예시 
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

#### 좋은점

- 코드 깔끔하다.
- 오류 가능성 적음
- 컬렉션, 배열이냐에 따라 코드형태 동일

#### 사용할 수 없는 상황

- 파괴적인 필터링
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야함
- 변형
    - 리스트, 배열 순회하면서 일부 값을 교체해야 한다면? 리스트의 반복자, 배열 인덱스 사용해야함
- 병렬 반복

<br/>

## 결론

- 가능한 모든 곳에서 for문이 아닌 for-each문 사용하자.