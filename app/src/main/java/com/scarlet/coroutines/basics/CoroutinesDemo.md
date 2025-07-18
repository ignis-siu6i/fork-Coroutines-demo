# CoroutinesDemo

## 1. 코루틴을 활용한 멀티태스킹과 yield

```kotlin
private suspend fun task1() {
    for (i in 1..10) {
        log("    Coroutine 1: $i")
        yield()
    }
}

runBlocking {
    val coroutine1 = launch { task1() }
    val coroutine2 = launch { task2() }
    val coroutine3 = launch { task3() }
    joinAll(coroutine1, coroutine2, coroutine3)
    log("Done!")
}
```
- **설명:**
  여러 코루틴이 번갈아가며 실행(yield)되는 멀티태스킹 예제입니다. yield는 다른 코루틴에 실행 기회를 양보합니다.

---

## 2. Sequence를 이용한 제너레이터 패턴

```kotlin
private fun fib(): Sequence<Int> = sequence {
    var previous = 0
    var current = 1
    while (true) {
        yield(previous)
        previous = current.also { current += previous }
    }
}
```
- **설명:**
  Kotlin의 Sequence와 yield를 이용해 제너레이터(이터레이터) 패턴을 구현할 수 있습니다. 필요할 때마다 값을 생성해 메모리 효율적입니다.

---

## 3. 결론 및 참고

- **코루틴과 yield, Sequence:**
  - 협력적 멀티태스킹, 게으른 계산, 반복자 패턴 등 다양한 활용 가능

- **참고 자료:**
  - [Kotlin 공식 문서: Sequences](https://kotlinlang.org/docs/sequences.html)
  - [Kotlin 공식 문서: yield](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/yield.html) 