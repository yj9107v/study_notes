# 📚 Chapter 5: File, Files

> 📌 공부 날짜: 2025/05/20
> - `JAVA `- 고급 2편
> - `References`: 인프런 - 김영한의 실전 자바

---

## ✅ File
- 자바에서 파일 또는 디렉터리를 다룰 때는 `File` 또는 `Files`, `Path` 클래스를 사용하면 된다.
- 이 클래스들을 사용하면 파일이나 폴더를 생성하고, 삭제하고, 또 정보를 확인할 수 있다.
- `File`은 파일과 디렉터리를 둘 다 다룬다.
  - 참고로 `File` 객체를 생성했다고 파일이나 디렉터리가 바로 만들어지는 것은 아니다. 메서드를 통해 생성해야 한다.
  ```java
  // ex)
  File file = new File("temp/example.txt");
  File directory = new File("temp/exampleDir");
  ```
- `File`과 같은 클래스들은 학습해야 할 중요한 원리가 있는 것이 아니라, 다양한 기능의 모음을 제공한다.
- 이런 클래스의 기능들은 외우기보다는 이런 것들이 있다 정도만 알아두고, 필요할 때 찾아서 사용하면 된다.

---

## ✅ Files
### 📚 File, Files 역사
- `File` 클래스를 대체할 `Files`와 `Path`가 등장했다.

#### 🔍 Files의 특징
- 성능과 편의성이 모두 개선되었다.
- `File`은 과거의 호환을 유지하기 위해 남겨둔 기능이다. 이제는 `Files` 사용을 먼저 고려하자.
- 여기에는 수많은 유틸리티 기능이 있다. `File` 클래스는 물론이고, `File`과 관련된 스트림(`FileInputStream`, `FileWriter`)의 사용을 고민하기 전에 `Files`에 있는 기능을 먼저 찾아보자.
  - 성능도 좋고, 사용하기도 더 편리하다.
- 이렇게 기능 위주의 클래스는 외우는 것이 아님. 주요 기능만 알아두고, 나머지는 필요할 때 검색하자.
- 참고로 `Files`를 사용할 때 파일이나, 디렉터리의 경로는 `Path` 클래스를 사용해야 한다.

```java
import org.w3c.dom.ls.LSOutput;

// ex)
Path file = Path.of("temp/example.txt");
Path directory = Path.of("temp/exampleDir");

try{
    Files.createFile(file);
} catch(FileAlreadyExistsException e) {
    System.out.println(file);
}
```
- `Files`는 직접 생성할 수 없고, static 메서드를 통해 기능을 제공한다.

### 📚 경로 표시
- 파일이나 디렉터리가 있는 경로는 크게 절대 경로와 정규 경로로 나눌 수 있다.
- **절대 경로(Absolute path)**: 절대 경로는 경로의 처음부터 내가 입력한 모든 경로를 다 표현한다.
- **정규 경로(Canonical path)**: 경로의 계산이 모두 끝난 경로이다. 정규 경로는 하나만 존재한다.
  - 예제에서 `..`은 바로 위의 상위 디렉터리를 뜻한다. 이런 경로의 계산을 모두 처리하면 하나의 경로만 남는다.
- 예를 들어서 절대 경로는 다음 2가지 경로 모두 가능
  - `Users/yh/study/inflearn`
  - `Users/yh/study/inflearn/temp/..`
- 정규 경로는 다음 하나만 가능
  - `Users/yh/study/inflearn`

---

### 📚 Files로 문자 파일 읽기
- 문자로 된 파일을 읽고 쓸 때 과거에는 `FileReader`, `FileWriter` 같은 복잡한 스트림 클래스를 사용해야 했다.
- 거기에 모든 문자를 읽으려면 반복문을 사용해서 파일의 끝까지 읽어야 하는 과정을 추가해야 한다.
- 또, 한 줄 단위로 파일을 읽으려면 `BufferedReader`와 같은 스트림 클래스를 추가해야 했다.
- **`Files`는 이런 문제를 코드 한 줄로 깔끔하게 해결해 준다.**

```java
// ex)
String writeString = "abc\n가나다";
Path path = Path.of("temp/hello2.txt");

Files.writeString(path, writeString, UTF_8); // 파일에 쓰기
String readString = Files.readString(path, UTF_8); // 파일에서 모든 문자 읽기
```

### 📚 Files - 라인 단위로 읽기
```java
// ex)
List<String> lines = Files.readAllLines(path, UTF_8);
for (int i = 0; i < lines.size(); i++) {
    System.out.println(lines.get(i));
}
```
#### 🔍 Files.readAllLines(path)
- 파일을 한 번에 다 읽고, 라인 단위로 `List`에 나누어 저장하고 반환한다.

##### 🔍 Files.lines(path)
- 파일을 한 줄 단위로 나누어 읽고, 메모리 사용량을 줄이고 싶다면 이 기능을 사용하면 된다.
```java
try (Stream<String> lineStream = Files.lines(path, UTF_8)) {
    lineStream.forEach(line -> System.out.println(line));
} 
 ```
- 파일을 스트림 단위로 나누어 조회한다.
- 파일이 아주 크다면 한 번에 모든 파일을 다 메모리에 올리는 것보다, 파일을 부분 부분 나누어 메모리에 올리는 것이 더 나은 선택일 수 있다.
- 예를 들어 파일의 크기가 1000MB라면 한 번에 1000MB의 파일을 메모리 불러오는 것보다 한 줄 단위로 메모리에 올리는 것이 좋다.
  - 한 줄 당 1MB의 용량을 사용한다면 자바는 파일에서 한 번에 1MB의 데이터만 메모리에 올려 처리하고, 기존에 사용한 1M의 데이터는 GC 한다.
- 용량이 아주 큰 파일을 처리해야 한다면, 이런 방식으로 나누어 처리하는 것이 효과적이다.
- 물론 `BufferedReader`를 통해서도 한 줄 단위로 이렇게 나누어 처리하는 것이 가능.
  - 여기서 핵심은 매우 편리하게 문자를 나누어 처리하는 것이 가능하다는 점이다.

---

## ✅ 파일 복사 최적화