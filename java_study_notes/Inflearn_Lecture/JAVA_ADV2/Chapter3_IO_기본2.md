# 📚 Chapter 3: IO 기본 2

> 📌 공부 날짜: 2025/05/17
> - `JAVA `- 고급 2편
> - `Reference`: 인프런 - 김영한의 실전 자바

---

## ✅ 문자 다루기 1 - 시작
- 스트림의 모든 데이터는 `byte` 단위를 사용한다. 따라서 `byte`가 아닌 문자를 스트림에 직접 전달할 수는 없다.
- 예를 들어서 `String` 문자를 스트림을 통해 파일에 저장하려면 `String`을 `byte`로 변환한 다음에 저장해야 한다.

```java
// ex)
String writeString = "ABC";
byte[] writeBytes = writeString.getBytes(UTF_8); // 파일에 쓰기 위해 byte로 인코딩.

byte[] readBytes = fis.readAlLBytes();
String readString = new String(readBytes, UTF_8); // 파일에서 읽은 byte를 String으로 디코딩
```

- `String`을 `byte`로 변환할 때는 `String.getBytes(Charset)`을 사용하면 된다.
  - 이때 문자를 `byte` 숫자로 변경해야 하기 때문에 반드시 **문자 집합(인코딩 셋)**을 지정해야 한다.
  - 여기서는 UTF_8로 인코딩 했다.
- 반대의 경우도 비슷하다. `String` 객체를 생성할 때, 읽어 들인 `byte[]`과 디코딩할 문자 집합을 전달하면 된다.
  - 그러면 `byte[]`를 `String` 문자로 다시 복원할 수 있다.
- **여기서 핵심은 스트림은 `byte`만 사용할 수 있으므로, `String`과 같은 문자는 직접 전달할 수는 없다는 점이다.**
  - 그래서 개발자가 번거롭게 변환 과정을 직접 호출해주어야 한다.
- 이렇게 번거운 변환 과정을 대신 처리해 주는 것이 필요해 보인다.

---

## ✅ 문자 다루기 2 - 스트림을 문자로
- **OutputStreamWriter**: 스트림에 byte 대신에 문자를 저장할 수 있게 해 준다.
- **InputStreamReader**: 스트림에 byte 대신에 문자를 읽을 수 있게 지원한다.

#### 🔍 OutputStreamWriter
- `OutputStreamWriter`는 문자를 입력받고, 받은 문자를 인코딩해서 `byte[]`로 변환한다.
- `OutputStreamWriter`는 변환한 `byte[]`을 전달할 `OutputStream`과 인코딩 문자 집합에 대한 정보가 필요하다.
  - 따라서 두 정보를 생성자를 통해 전달해야 한다.
  - `new OutputStreamWriter(fos, UTF_8)`
- `osw.write(writeString)`를 보면 `String` 문자를 직접 전달하는 것을 확인할 수 있다.

#### 🔍 InputStreamReader
```java
FileInputStream fis = new FileInputStream(FILE_NAME);
InputStreamReader isr = new InputStreamReader(fis, UTF_8);

StringBuilder sb = new StringBuilder();
int ch;
while ((ch = isr.read()) != -1) {
    sb.append((char) ch);
}
isr.close();
```
- 데이터를 읽을 때는 `int ch = read()`를 제공하는데, 여기서는 문자 하나인 `char`형으로 데이터를 받게 된다.
  - 그런데 실제 반환 타입은 `int`형이므로 `char`형으로 캐스팅해서 사용하면 된다.
- 자바의 `char`형은 파일의 끝인 `-1`을 표현할 수 없으므로 대신 `int`를 반환한다.
- `InputStreamReader`는 이렇게 읽은 `byte[]`을 문자인 `char`로 변경해서 반환한다.
  - 물론 `byte`를 문자로 변경할 때도 문자 집합이 필요하다.
  - `new InputStreamReader(fis, UTF_8)`

---

## ✅ 문자 다루기 3 - Reader, Writer
- 자바는 byte를 다루는 I/O 클래스와 문자를 다루는 I/O 클래스를 둘로 나누어 두었다.

#### 🔍 byte를 다루는 클래스
- `OutputStream`, `InputStream`의 자식이다.
  - 부모 클래스는 기본 기능도 `byte` 단위를 다룬다.
  - 클래스 이름 마지막에 보통 `OutputStream`, `InputStream`이 붙어있다.

#### 🔍 문자를 다루는 클래스
- `Writer`, `Reader`의 자식이다,
  - 부모 클래스의 기본 기능은 `String`, `char` 같은 문자를 다룬다.
  - 클래스 이름 마지막에 보통 `Writer`, `Reader`가 붙어있다.

> ❗❗ 여기서 **꼭! 기억해야 할 중요한 사실이 있다. 처음에 언급했듯이 모든 데이토는 byte 단위(숫자)로 저장된다.**
> - 따라서 `Writer`가 아무리 문자를 다룬다고 해도 문자를 바로 저장할 수는 없다.
> - 이 클래스에 문자를 전달하면 결과적으로 내부에서는 지정된 문자 집합을 사용해서 문자를 byte로 인코딩해서 저장한다.

### 📚 FileWriter, FileReader
```java
//ex)
String writeString = "ABC";
FileWriter fw = new FileWriter(FILE_NAME, UTF_8);
fw.write(writeString);
fw.close();

StringBuilder sb = new StringBuilder();
FildReader fr = new FileReader(FILE_NAME, UTF_8);
int ch;
while ((ch = fr.read()) != -1) {
    sb.append((char) ch);    
}
fr.close();
```
- `FileWriter`에 파일명과, 문자 집합(인코딩 셋)을 전달한다.
- `FileWriter`는 사실 내부에서 스스로 `FileOutputStream`을 하나 생성해서 사용한다.
  - 모든 데이터는 byte 단위로 저장된다는 사실을 다시 떠올리자.
- `FileReader` 또한 마찬가지이다.

#### 🔍 FileWriter와 OutputStreamWriter
- `FileWriter` 코드와 앞서 작성한 `OutputStreamWriter`를 사용한 코드가 뭔가 비슷하다는 점을 알 수 있다.
- 딱 하나 차이점이 있다면 이전 코드에서는 `FileOutputStream`을 직접 생성했는데, `FileWriter`는 생성자 내부에서 대신 `FileOutputStream`를 생성해 준다.
- 사실 `FileWriter`는 `OutputStreamWriter`을 상속한다. 그리고 다른 추가 기능도 없다.
- `FileReader`도 마찬가지다. 개발자가 조금 더 편리하게 사용하도록 도와줄 뿐이다.

### 📌 정리
- `Writer`, `Reader` 클래스를 사용하면 바이트 변환 없이 문자를 직접 다룰 수 있어서 편리하다.
- 하지만 실제로는 내부에서 byte로 변환해서 저장한다는 점을 꼭 기억하자.
- 모든 데이터는 바이트 단위로 다룬다! 문자를 직접 저장할 수는 없다!
- 그리고 반드시 기억하자, 문자를 byte로 변경하려면 항상 문자 집합(인코딩 셋)이 필요하다!


- **참고: 문자 집합을 생략하면 시스템 기본 문자 집합이 사용된다.**

---

## ✅ 문자 다루기 4 - BufferedReader
- `BufferedOutputStream`, `BufferedInputStream`과 같이 `Reader`, `Writer`에도 버퍼 보조 기능을 제공하는 `BufferedWriter`, `BufferedReader` 클래스가 있다.
- 추가로 문자를 다룰 때는 한 줄(라인) 단위로 다룰 때가 많다.
- `BufferedReader`는 한 줄 단위로 문자를 읽는 기능도 추가로 제공한다.

#### 🔍 br.readLine()
- 한 줄 단위로 문자를 읽고 `String`을 반환한다.
- 파일의 끝(EOF)에 도달하면 `null`을 반환한다.
  - 반환 타입이 `String`이기 때문에 EOF를 -1로 표현할 수 없다. 대신에 `null`을 반환한다.

---

## ✅ 기타 스트림
### 📚 PrintStream
-`PrintStream`은 우리가 자주 사용했던 바로 `System.out`에서 사용되는 스트림이다.
`PrintStream`과 `FileOutputStream`을 조합하면 마치 콘솔에 출력하듯이 파일에 출력할 수 있다.

```java
// ex)
FileOutputStream fos = new FileOutputStream("temp/print.txt");
PrintStream ps = new PrintStream(fos);
ps.println("hello java!");
```

- `PrintStream`의 생성자에 `FileOutputStream`을 전달했다. 이제 이 스트림을 통해서 나가는 출력은 파일에 저장된다.
- 이 기능을 사용하면 마치 콘솔에 출력하는 것처럼 파일이나 다른 스트림에 문자를 출력할 수 있다.

---

### 📚 DataOutputStream
- `DataOutputStream`을 사용하면 자바의 `String`, `int`, `double`, `boolean` 같은 데이터 형을 편리하게 다룰 수 있다.
- 이 스트림과 `FileOutputStream`을 조합하면 파일에 자바 데이터 형을 편리하게 저장할 수 있다.

```java
// ex)
DataOutputStream dos = new DataOutputStream(fos);

dos.writeUTF("회원A");
dos.writeInt(20);
dos.writeDouble(10.5);
dos.writeBoolean(true);
dos.close();
```

- 예제를 보면 자바 데이터 타입을 사용하면서, 회원 데이터를 편리하게 저장하고 불러오는 것을 확인할 수 있다.
- ⚠️ 이 스트림을 사용할 때 주의할 점으로는 꼭! 저장한 순서대로 읽어야 한다는 것이다.
  - 그렇지 않으면 잘못된 데이터가 조회될 수 있다.


- 저장한 `data.dat` 파일을 직접 열어보면 제대로 보이지 않는다.
  - 왜냐하면 `writeUTF()`의 경우 UTF-8 형식으로 저장하지만, 나머지의 경우 문자가 아니라 각 타입에 맞는 byte 단위로 저장하기 때문이다.
- 예를 들어서 자바에서는 `int`는 4byte를 묶어서 사용한다. 해당 byte가 그대로 저장되는 것이다.
- 텍스트 편집기는 자신의 문자 집합을 사용해서 byte를 문자로 표현하려고 시도하지만 문자 집합에 없는 단어이거나 또는 전혀 예상하지 않은 문자로 디코딩 될 것이다.

---

## 📌 정리
- 기본(기반, 메인) 스트림
  - File, 메모리, 콘솔 등에 직접 접근하는 스트림
  - 단독으로 사용할 수 있음
  - 예) `FileInputStream`, `FileReader`, `ByteArrayInputStream` -> 반대인 `Output`도 있음.
- 보조 스트림
  - 기본 스트림을 도와주는 스트림
  - 단독으로 사용할 수 없음. 반드시 대상 스트림이 있어야 함.
  - 예) `BufferedInputStream`, `InputStreamReader`, `DataOutputStream`, `PrintStream`

---