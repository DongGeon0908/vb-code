# VB Code

> 데이터 압축에서 사용되는 기법 중 하나로, 주로 정수 값을 효율적으로 저장하기 위해 사용한다.
> 정보 검색 시스템에서 인덱스 파일의 크기를 줄이기 위해 흔히 사용되며, 가변 길이 인코딩 기법의 한 종류로, 작은 값일수록 적은 바이트를 사용하여 저장할 수 있게 설계되어있다.

### 원리

> Variable-Byte Code는 정수를 여러 바이트로 분할하여 저장하며, 각 바이트의 최상위 비트를 사용하여 다음 바이트가 존재하는지 여부를 표시한다.
> 최상위 비트가 1이면 이어지는 바이트가 더 있다는 의미이고, 0이면 현재 바이트가 마지막이라는 의미한다. 나머지 7비트는 실제 데이터 값을 저장한다.

### 예시

**상황** 

300이라는 숫자를 인코딩하려고 한다면:

- 300은 이진수로 100101100입니다.
- 이를 7비트 단위로 나누면 0000010과 0101100으로 분할할 수 있습니다.
- 첫 번째 바이트는 더 큰 숫자를 표현할 필요가 없으므로, 0101100에 0을 붙여 00101100이 되고,
- 두 번째 바이트는 뒤에 값이 더 있다는 것을 나타내기 위해 0000010에 1을 붙여 10000010가 됩니다.
- 최종적으로 이 숫자는 10000010 00101100으로 인코딩됩니다.

### 장점

- 효율성: 작은 숫자는 적은 바이트로, 큰 숫자는 더 많은 바이트를 사용하므로, 작은 값이 많이 사용되는 상황에서 매우 효율적입니다.
- 단순성: 구현이 비교적 간단하며, 인코딩과 디코딩 과정이 빠릅니다.

### 단점
- 가변 길이: 고정 길이 인코딩에 비해 다루기 복잡할 수 있으며, 접근 속도가 느려질 수 있습니다.
- 메모리 사용: 큰 숫자를 표현할 때는 여러 바이트를 사용하게 되어, 고정 길이 인코딩에 비해 메모리 사용량이 증가할 수 있습니다.

### Example Code

```kotlin
object VBCode {
    fun encodeNumber(number: Int): ByteArray {
        val bytes = mutableListOf<Byte>()
        var n = number

        while (true) {
            /** 하위 7비트 추출 */
            var byte = (n and 0x7F).toByte()
            n = n shr 7
            if (n == 0) {
                bytes.add(byte)
                break
            } else {
                /** 다음 바이트가 있음을 표시 */
                byte = (byte.toInt() or 0x80).toByte()
                bytes.add(byte)
            }
        }

        return bytes.toByteArray()
    }

    fun decodeBytes(bytes: ByteArray): Int {
        var number = 0
        var shift = 0

        for (byte in bytes) {
            number = number or ((byte.toInt() and 0x7F) shl shift)
            if (byte.toInt() and 0x80 == 0) {
                break
            }
            shift += 7
        }

        return number
    }

    fun encode(numbers: List<Int>): ByteArray {
        val encodedBytes = mutableListOf<Byte>()
        numbers.forEach { number ->
            encodedBytes.addAll(encodeNumber(number).toList())
        }
        return encodedBytes.toByteArray()
    }

    fun decode(bytes: ByteArray): List<Int> {
        val numbers = mutableListOf<Int>()
        val tempBuffer = mutableListOf<Byte>()

        bytes.forEach { byte ->
            tempBuffer.add(byte)

            if (byte.toInt() and 0x80 == 0) {
                numbers.add(decodeBytes(tempBuffer.toByteArray()))
                tempBuffer.clear()
            }
        }

        return numbers
    }
}

/** test */
fun main() {
    val numbers = listOf(300, 400, 500)
    println("Original Numbers: $numbers")

    val encoded = VBCode.encode(numbers)
    println("Encoded Bytes: ${encoded.joinToString { it.toString() }}")

    val decoded = VBCode.decode(encoded)
    println("Decoded Numbers: $decoded")
}
```

### Reference

- [stanford.edu](https://nlp.stanford.edu/IR-book/html/htmledition/variable-byte-codes-1.html)
