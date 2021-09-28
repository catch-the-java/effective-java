# :bulb: try-with-resource
자바 라이브러리에서 close 메서드를 호출해서 직접 닫아줘야 하는 자원이 많다. 직접 close를 작성하는 대신에 try-with-resource를 사용해보도록 하자.


## try-finally
  - 자원이 2개 이상이 생기면 try-finally 방식은 너무 지저분해 진다.
  - try 블록과 finally 블록 모두에서 예외가 발생할 수 있다. ( 디버깅이 어렵다. )

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
	}
```

## try-with-resource
- AutoCloseable 인터페이스를 구현해야 한다. 
  - 이미 많은 라이브러리나 클래스가 AutoCloseable을 구현하거나 확장했다.
  - 우리가 개발할 때, 닫아야 하는 자원을 뜻하는 클래스를 만든다면 **AutoCloseable**을 구현하자. 

- InputStream에 Closeable을 implements 한 것을 확인할 수 있다. 
```java
public abstract class InputStream implements Closeable {
  .
  .

  public void close() throws IOException {}
}
```

- 장점
  1. 짧고 읽기 쉽다. 
  2. 문제를 진단하기에 좋다. 
- 사용 예시 
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
- catch절도 사용할 수 있다.