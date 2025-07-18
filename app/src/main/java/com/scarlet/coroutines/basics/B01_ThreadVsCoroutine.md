# B01_ThreadVsCoroutine

이 파일은 스레드와 코루틴의 동작 방식, 성능, 자원 사용, 코드 구조의 차이를 실험적으로 비교합니다. 각 main 함수는 실제로 실행 가능한 실험 코드로, 코루틴의 경량성, 효율성, 그리고 실무에서의 장점을 직접 확인할 수 있게 설계되어 있습니다.

---

## Threads.main
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
```
- **설명:**
  - 10만 개의 OS 스레드를 생성해 1초씩 sleep합니다. 각 스레드는 "."을 출력합니다.
  - 모든 스레드가 종료될 때까지 join으로 대기합니다.
  - **실행 결과:** 대부분의 환경에서 OutOfMemoryError가 발생하거나, 시스템이 매우 느려집니다. 스레드는 무겁고, 대량 생성에 적합하지 않음을 보여줍니다.
  - **의도:** 코루틴의 경량성과 비교하기 위한 극단적 실험입니다.

---

## Coroutines.main
```kotlin
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
  - 10만 개의 코루틴을 launch로 생성해 1초씩 delay합니다. 각 코루틴은 "."을 출력합니다.
  - 모든 코루틴이 종료될 때까지 join으로 대기합니다.
  - **실행 결과:** OutOfMemory 없이 정상적으로 동작하며, 스레드와 달리 시스템 자원을 거의 소모하지 않습니다.
  - **의도:** 코루틴이 스레드보다 훨씬 경량적임을 실험적으로 보여줍니다.

---

## ThreadVsCoroutine.main
```kotlin
object ThreadVsCoroutine {
    @DelicateCoroutinesApi
    @JvmStatic
    fun main(args: Array<String>) {
        threads()
//        coroutines()
    }
    // ...
}
```
- **설명:**
  - threads()와 coroutines() 두 가지 실험을 선택적으로 실행할 수 있습니다.
  - **threads()**: 메인 스레드와 별도의 백그라운드 스레드를 생성해, 백그라운드 작업이 끝날 때까지 메인 스레드가 기다리지 않음을 보여줍니다.
    - 로그 예시:
      - Main program starts: main
      - Background work starts: Thread-0
      - Main program ends: main
      - Background work ends: Thread-0
  - **coroutines()**: GlobalScope.launch로 백그라운드 코루틴을 생성합니다. 메인 스레드는 코루틴의 완료를 기다리지 않고 바로 종료됩니다(주석 처리된 Thread.sleep을 해제하면 대기 가능).
    - 로그 예시:
      - Main program starts: main
      - Main program ends: main
      - Background work starts: main (코루틴은 기본적으로 main 스레드에서 실행됨)
      - Background work ends: main
  - **의도:** 스레드와 코루틴의 실행 흐름, 동기/비동기 차이, 자원 사용, 코드 구조의 차이를 비교합니다.

---

## 결론 및 참고
- **코루틴의 장점:**
  - 경량성: 수만 개의 코루틴도 문제없이 동작
  - 자원 효율성: 스레드보다 훨씬 적은 메모리 사용
  - 코드의 간결함과 유지보수성
- **참고 자료:**
  - [Kotlin 공식 문서: Coroutines vs Threads](https://kotlinlang.org/docs/coroutines-basics.html#coroutines-vs-threads) 