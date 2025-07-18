# TrampolineDemo

## 1. 재귀, 꼬리재귀, 트램폴린 패턴

```kotlin
fun factorial(n: Long): BigInteger =
    if (n <= 1) BigInteger.ONE else n.toBigInteger() * factorial(n - 1)

tailrec fun factorialTR(n: Long, acc: BigInteger): BigInteger =
    if (n <= 1) acc else factorialTR(n - 1, acc * n.toBigInteger())
```
- **설명:**
  일반 재귀는 스택 오버플로우 위험이 있지만, 꼬리재귀(tailrec)는 컴파일러가 반복문으로 최적화해줍니다. 트램폴린(trampoline) 패턴은 더 복잡한 재귀를 안전하게 처리할 수 있게 해줍니다.

---

## 2. 트램폴린 패턴의 개념

- **설명:**
  트램폴린은 재귀 호출을 명시적 스택(함수 객체 등)으로 변환해, JVM의 콜스택을 사용하지 않고도 깊은 재귀를 안전하게 처리할 수 있게 해줍니다. lazy evaluation과 결합해 무한 재귀도 안전하게 구현할 수 있습니다.

---

## 3. 결론 및 참고

- **트램폴린/꼬리재귀의 활용:**
  - 스택 오버플로우 방지, 함수형 프로그래밍, lazy evaluation 등

- **참고 자료:**
  - [Kotlin 공식 문서: tailrec](https://kotlinlang.org/docs/functions.html#tail-recursive-functions)
  - [트램폴린 패턴 설명(영문)](https://medium.com/@kasperpeulen/what-is-a-trampoline-function-27c2f4891b0a) 