# B01_ThreadVsCoroutine

## 1. 스레드와 코루틴의 대량 생성 비교

```kotlin
object Threads {
    @JvmStatic
    fun main(args: Array<String>) {
        val time = measureTimeMillis {
            val jobs = List(100_000) {
                thread {
                    print(".")
                    Thread.sleep(1_000)
                }
            }
            jobs.forEach { it.join() }
        }
        println("\nElapses time = $time ms")
    }
}

object Coroutines {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val time = measureTimeMillis {
            val jobs = List(100_000) {
                launch {
                    print(".")
                    delay(1_000)
                }
            }
            jobs.forEach { it.join() }
        }
        println("\nElapses time = $time ms")
    }
}
```
- **설명:**
  10만 개의 스레드와 코루틴을 각각 생성해 실행하는 실험입니다. 스레드는 OS 자원을 많이 소모해 OutOfMemory가 발생할 수 있지만, 코루틴은 경량 스레드로 수만 개도 무리 없이 동작합니다.

---

## 2. 메인 스레드와 백그라운드 작업 비교

```kotlin
private fun threads() {
    log("Main program starts: ${'$'}{Thread.currentThread().name}")
    thread {
        log("Background work starts: ${'$'}{Thread.currentThread().name}")
        Thread.sleep(1_000)
        log("Background work ends: ${'$'}{Thread.currentThread().name}")
    }
    log("Main program ends: ${'$'}{Thread.currentThread().name}")
}

private fun coroutines() {
    log("Main program starts: ${'$'}{Thread.currentThread().name}")
    GlobalScope.launch {
        log("Background work starts: ${'$'}{Thread.currentThread().name}")
        delay(1_000)
        log("Background work ends: ${'$'}{Thread.currentThread().name}")
    }
    log("Main program ends: ${'$'}{Thread.currentThread().name}")
}
```
- **설명:**
  스레드와 코루틴 모두 백그라운드 작업을 실행할 수 있지만, 코루틴은 더 적은 자원으로 더 많은 동시 작업을 처리할 수 있습니다.

---

## 3. 결론 및 참고

- **코루틴의 장점:**
  - 경량성: 수만 개의 코루틴도 문제없이 동작
  - 자원 효율성: 스레드보다 훨씬 적은 메모리 사용
  - 코드의 간결함과 유지보수성

- **참고 자료:**
  - [Kotlin 공식 문서: Coroutines vs Threads](https://kotlinlang.org/docs/coroutines-basics.html#coroutines-vs-threads) 