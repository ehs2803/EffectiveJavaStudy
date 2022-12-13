# item 9. try-finally 보다 try-with-resources를 사용하라.
try-with-resources는 try블록이 끝나면 자원을 반납하는 기능을 가진다. 
try구문안에 전달할 수 있는 자원은 AutoCloseable인터페이스 구현체로 제한된다. 이로써 finally 구문에서 자원을 반납하지 않아도 된다.
![image](https://user-images.githubusercontent.com/45592236/207356621-9403cddc-d0ac-433d-95e5-d4f8eed7ca02.png)

try-finally를 이용하여 exception 발생 순서를 확인하고자 한다.

```java
static String firstLineOfFile(String path) throws IOException {
		BadBufferedReader br = new BadBufferedReader(new FileReader(path));
		try{
			return br.readLine();
		} finally {
			br.close();
		}
}

public static void main(String[] args) throws IOException {
    String path = "application.properties";
    System.out.println("firstLineOfFiel(path) = " + firstLineOfFile(path));
}
```
BuffereadReader를 오버라이딩 하였다.
```java
public class BadBufferedReader extends BufferedReader {
	public BadBufferedReader(Reader in, int sz) {
		super(in, sz);
	}

	public BadBufferedReader(Reader in) {
		super(in);
	}

	@Override
	public String readLine() throws IOException {
		throw new CharConversionException();
	}

	@Override
	public void close() throws IOException {
		throw new StreamCorruptedException();
	}
}
```

```java
Exception in thread "main" java.io.StreamCorruptedException
	at BadBufferedReader.close(BadBufferedReader.java:28)
	at Item9.firstLineOfFile(Item9.java:41)
	at Item9.main(Item9.java:47)
```

디버깅 시 제일 처음 발생한 exception을 확인해야하므로 중요한데, 맨 처음 발생한 exception이 찍히는것이 아닌 가장 마지막에 발생한 Exception이 stack trace에 찍히게 된다.

위의 문제는 java7에서 등장한 try-with-resource를 통해 해결된다.

```java
static String firstLineOfFile(String path) throws IOException {
		try(BufferedReader br = new BufferedReader(new FileReader(path))) {//resource block setting
			return br.readLine();
		}
}

public static void main(String[] args) throws IOException {
    String path = args[0];
    System.out.println("firstLineOfFiel(path) = " + firstLineOfFile(path));
}
```

try-with-resource 구문을 통해 exception 발생 순서를 확인할 수 있다.

```java
Exception in thread "main" java.io.CharConversionException
	at BadBufferedReader.readLine(BadBufferedReader.java:23)
	at Item9.firstLineOfFile(Item9.java:38)
	at Item9.main(Item9.java:44)
	Suppressed: java.io.StreamCorruptedException
		at BadBufferedReader.close(BadBufferedReader.java:28)
		at Item9.firstLineOfFile(Item9.java:37)
		... 1 more
```
try-with-resource구문을 사용하면 여러개의 자원을 try구문에서 깔끔하게 작성이 가능하다.
코드도 깔금해지고 자원이 닫히는것이 보장된다.
```java
static void firstLineOfFile(String path, String path2) throws IOException {
    try(InputStream in = new FileInputStream(path);
        OutputStream out = new FileOutputStream(path2)) {//resource block setting
        byte[] buf = new byte[8 * 1024];
        int n;
        while((n = in.read(buf)) >= 0)
            out.write(buf, 0,n);

    }
}

public static void main(String[] args) throws IOException{
	String path=args[0];
	String path2=args[1];
	firstLineOfFile(path,path2);
}
	
```
