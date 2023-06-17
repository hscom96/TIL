# 아이템 09. try-finally 보다는 try-with-resources를 사용하라

## 사용 배경

- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아야 하는 자원이 많다.
- 하지만 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어진다.

<br/>

## try-finally

```java
static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    } sssss
```

- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 사용
- __문제점__
    - 코드가 너무 지저분하다. (특히 자원 둘이상이면..)
    - close 메서드 호출을 잊어버릴 수 있다.
    - 예외는 try블록, finally 블록 모두에서 발생
        - close 메서드 호출 실패할 수 있다.
        - 두번째 예외가 첫번째 예외를 삼켜버려 디버깅 어려워짐

<br/>

## try-with-resources

위 try-finally의 문제점을 try-with-resource가 모두 해결했다.

```java
static void copy(String src, String dst) throws IOException {
		try (InputStream in = new FileInputStream(src);
		     OutputStream out = new FileOutputStream(dst)) {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		}
	}
```

- 이 구조를 사용하려면 __자원이 AutoCloseable 인터페이스를 구현해야한다.__

<br/>

- 장점
    - 코드 짧고 간결해짐
        - try문을 더 중첩하지 않고 다수의 예외를 처리가능
    - 정확하고 쉽게 자원회수 가능
