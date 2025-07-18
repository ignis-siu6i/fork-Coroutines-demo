# CoroutinesDemo

이 파일은 코루틴을 활용한 멀티태스킹, yield를 통한 협력적 스케줄링, Sequence를 이용한 제너레이터 패턴 등 코루틴의 기본적인 활용법을 예제로 설명합니다.

---

## Coroutines_Multitasking.main
```kotlin
object Coroutines_Multitasking {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val coroutine1 = launch { task1() }
        val coroutine2 = launch { task2() }
        val coroutine3 = launch { task3() }
        joinAll(coroutine1, coroutine2, coroutine3)
        log("Done!")
    }
}
```
- **설명:**
  - 여러 코루틴을 launch로 동시에 실행하고, 각 코루틴은 yield를 통해 실행을 양보합니다.
  - log를 통해 여러 코루틴이 번갈아가며 실행되는 협력적 멀티태스킹을 확인할 수 있습니다.
  - **의도:** yield의 동작과 멀티태스킹, 코루틴 간 실행 양보의 효과를 실험적으로 보여줍니다.

---

## GeneratorUsingSequence.main
```kotlin
object GeneratorUsingSequence {
    @JvmStatic
    fun main(args: Array<String>) {
        val iterator = fib().iterator()
        while (prompt()) {
            println("Got result = ${iterator.next()}")
        }
    }
}
```
- **설명:**
  - Sequence와 yield를 이용해 제너레이터(이터레이터) 패턴을 구현합니다.
  - 사용자가 next?를 입력할 때마다 피보나치 수열의 다음 값을 생성합니다.
  - **의도:** 게으른 계산(lazy evaluation)과 반복자 패턴, yield의 활용을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **코루틴과 yield, Sequence:**
  - 협력적 멀티태스킹, 게으른 계산, 반복자 패턴 등 다양한 활용 가능
- **참고 자료:**
  - [Kotlin 공식 문서: Sequences](https://kotlinlang.org/docs/sequences.html)
  - [Kotlin 공식 문서: yield](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/yield.html) 