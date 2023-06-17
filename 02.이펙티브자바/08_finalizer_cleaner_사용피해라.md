# 아이템 08. finalizer와 cleaner 사용을 피하라

## finalizer & cleaner 란?

- 자바는 객체 소멸자는 2가지 (finalizer과 그 대안인 cleaner)을 제공하지만 기본적으로는 사용하면 안된다.

> c++ destructor과 다르게 java에서는 객체를 회수하는 역할을 가비지 컬렉터가 담당.

<br/>

## 문제점

- __finalizer, cleaner는 실행되기까지 얼마나 걸릴지 알 수 없다.__
    - __심지어 수행 여부도 보장안한다.__
    - 즉, 제때 실행되어야 하는 작업은 할 수 없어서 문제 생길 수 있다.
    - 예를들어
        - 파일 닫기를 finalize, cleaner하면 처리를 게을리해서 오류 생길 수 있다. 시스템에 열 수 있는 파일 개수에는 한계가 있기 때문.
        - 클래스에 finalizer를 달아두면 인스턴스의 자원 회수가 늦어질 수도 있음. -> out of memory 위험
        - 상태를 영구적으로 수정하는 작업에서는 절대 finalizer, cleaner에 의존해서는 안된다.
            - ex) 데이터베이스 같은 공유 자원의 영구 락 해제

<br/>

- __finalize, cleaner는 심각한 성능, 보안문제 동반한다.__
    - finalizer는 가비지 컬렉터의 효율을 떨어 뜨린다.

<br/>

## 쓰임새

- finalizer, cleaner의 쓰임새는 두가지있다.

__1. 자원의 소유자가 close 메서드를 호출하지 않는 경우.__

- 즉시 호출되는 보장은 없지만, 아예 자원회수를 안하는것보다 늦게라도 하는 것이 좋다. (안정망 역할)
- 자바 라이브러리 일부는 안전망 역할의 finalizer 제공
    - ex. FileInputStream, FileOutputStream, ThreadPoolExecutor

__2. native peer와 연결된 객체에 처리하는 경우.__

- native peer는 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.
- 네이티브 피어는 자바객체가 아니여서 gc가 존재를 모르고 회수를 못한다.
- 단, 성능 저하를 감당할수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수 해야한다면 close 메서드 사용

__이 두가지 경우에도 불확실성, 성능저하에 주의__

<br/>

## 자원 반납 해결책

1. Autocloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고나면 close 메서드를 호출한다.
2. 일반적으로 예외가 발생해도 제대로 종료 되도록 try-with-resource 사용
