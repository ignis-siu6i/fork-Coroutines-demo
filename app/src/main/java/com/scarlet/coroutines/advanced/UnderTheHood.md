# UnderTheHood

## 1. suspend 함수의 내부 동작

```kotlin
suspend fun foo(arg: Int): Double {
    var local1 = 1.0
    var local2 = 2.0
    val res1 = bar(arg, local1)
    val res2 = bar(arg + 1, local2 + 1.0)
    return res1 + res2
}

suspend fun bar(m: Int, n: Double): Double {
    delay(1_000)
    return m + n
}
```
- **설명:**
  suspend 함수는 컴파일 시 상태 머신으로 변환되어, 중단점마다 현재 상태와 지역 변수를 저장합니다. delay 등 일시 중단 지점에서 코루틴이 중단되고, 재개 시 이어서 실행됩니다.

---

## 2. 상태 머신 변환과 레이블

- **설명:**
  컴파일러는 suspend 함수 내의 각 일시 중단 지점에 레이블을 붙여, 재개 시 어디서부터 실행할지 관리합니다. 이로 인해 스택 오버플로우 없이 깊은 비동기 체인을 구현할 수 있습니다.

---

## 3. 결론 및 참고

- **코루틴의 내부 동작 이해:**
  - 효율적인 비동기 처리, 스택 보존 없이 이어서 실행 가능

- **참고 자료:**
  - [Kotlin 공식 문서: Coroutines Under the Hood](https://kotlinlang.org/docs/coroutines-intrinsics.html)
  - [Kotlin Coroutines: Under the hood (블로그)](https://elizarov.medium.com/kotlin-coroutines-under-the-hood-4868d7abfc20) 