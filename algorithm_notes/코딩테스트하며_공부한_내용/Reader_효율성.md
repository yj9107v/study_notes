# ✅ Reader 기반 입력 처리가 효율적일 때 - 사용자 정의 클래스
### 🔍 `References`: GPT

---

## 📚 대량 입력이 있는 환경에서 Scanner, BufferedReader보다 훨씬 빠른 이유
- 사용자 정의로 구현한 빠른 입력 클래스이다.

### 🔍 전체 구조
```java
static class Reader {
    final int SIZE = 1 << 13; // 2^13 = 8192 bytes (8KB)
    byte[] buffer = new byte[SIZE]; // 입력을 저장할 버퍼
    int index, size;

    int nextInt() throws Exception {
        int n = 0;
        byte c;
        boolean isMinus = false;

        while ((c = read()) <= 32) { // 공백, 개행 등 무시
            if (size < 0) return -1;
        }

        if (c == 45) { // 음수 처리
            c = read();
            isMinus = true;
        }

        do {
            n = (n << 3) + (n << 1) + (c & 15); // n * 10 + (c - '0')
        } while (isNumber(c = read()));

        return isMinus ? ~n + 1 : n; // 음수 처리
    }

    boolean isNumber(byte c) {
        return 47 < c && c < 58; // '0' ~ '9'
    }

    byte read() throws Exception {
        if (index == size) {
            size = System.in.read(buffer, index = 0, SIZE);
            if (size < 0) buffer[0] = -1;
        }
        return buffer[index++];
    }
}

```

### 🧠 주요 구성 요소 설명
1. final int size = 1 << 13; (8KB)
    - 한 번에 읽어올 버퍼 크기를 8KB로 설정.
    - 1 << 13 = 2의 13승 = 8192 바이트.
    > Java에서는 System.in.read()는 한 바이트씩 읽으면 매우 느리다.
    > - 그래서 버퍼를 한 번에 크게 읽고, byte[] 배열로 관리하면 속도가 급격히 빨라진다.