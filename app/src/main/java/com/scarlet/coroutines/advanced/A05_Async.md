# A05_Async

이 파일은 async와 Deferred를 사용한 비동기 프로그래밍을 설명합니다. 병렬 실행, 순차 실행과의 차이, 지연 시작(CoroutineStart.LAZY), 예외 처리 등 다양한 async 활용 패턴을 예제로 보여줍니다.

---

## Async_Sequential_Demo.main
```kotlin
object Async_Sequential_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val time = measureTimeMillis {
            val one = doSomethingUsefulOne()
            val two = doSomethingUsefulTwo()
            log("The answer is ${one + two}")
        }
        log("Completed in $time ms")
    }
}
```
- **설명:**
  - 두 개의 suspend 함수를 순차적으로 실행하는 기본적인 방법입니다.
  - 첫 번째 함수가 완료된 후 두 번째 함수가 시작됩니다.
  - **의도:** async와 비교하기 위한 기준점으로, 순차 실행의 성능 특성을 실험적으로 보여줍니다.

---

## Async_Concurrent_Demo.main
```kotlin
object Async_Concurrent_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val time = measureTimeMillis {
            val one = async { doSomethingUsefulOne() }
            val two = async { doSomethingUsefulTwo() }
            log("The answer is ${one.await() + two.await()}")
        }
        log("Completed in $time ms")
    }
}
```
- **설명:**
  - async를 사용해 두 함수를 병렬로 실행하고 결과를 조합합니다.
  - 두 작업이 동시에 시작되어 총 실행 시간이 크게 줄어듭니다.
  - **의도:** async의 병렬 실행 능력과 성능 향상을 실험적으로 보여줍니다.

---

## Async_Lazy_Demo.main
```kotlin
object Async_Lazy_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val time = measureTimeMillis {
            val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
            val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
            
            one.start() // 첫 번째 시작
            two.start() // 두 번째 시작
            log("The answer is ${one.await() + two.await()}")
        }
        log("Completed in $time ms")
    }
}
```
- **설명:**
  - CoroutineStart.LAZY로 지연 시작을 설정하고 명시적으로 start()를 호출합니다.
  - 코루틴 생성과 실행 시점을 분리할 수 있습니다.
  - **의도:** 지연 시작을 통한 세밀한 실행 제어를 실험적으로 보여줍니다.

---

## Async_Exception_Demo.main
```kotlin
object Async_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferred = async {
            throw RuntimeException("Something went wrong")
        }
        
        try {
            deferred.await()
        } catch (e: RuntimeException) {
            log("Caught: $e")
        }
    }
}
```
- **설명:**
  - async에서 발생한 예외는 await() 호출 시점에 다시 던져집니다.
  - try-catch로 예외를 잡을 수 있습니다.
  - **의도:** async의 예외 처리 메커니즘과 지연된 예외 전파를 실험적으로 보여줍니다.

---

## Async_Cancellation_Demo.main
```kotlin
object Async_Cancellation_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferred1 = async {
            delay(1000)
            "Result 1"
        }
        
        val deferred2 = async {
            delay(2000)
            "Result 2"
        }
        
        delay(500)
        deferred1.cancel()
        
        try {
            log("Result: ${deferred1.await()}")
        } catch (e: CancellationException) {
            log("deferred1 was cancelled")
        }
        
        log("deferred2 result: ${deferred2.await()}")
    }
}
```
- **설명:**
  - async로 생성된 Deferred도 취소할 수 있습니다.
  - 취소된 Deferred의 await()은 CancellationException을 던집니다.
  - **의도:** async의 취소 메커니즘과 부분적 취소를 실험적으로 보여줍니다.

---

## Async_awaitAll_Demo.main
```kotlin
object Async_awaitAll_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferreds = (1..3).map { i ->
            async {
                delay(i * 100L)
                i * i
            }
        }
        
        val results = deferreds.awaitAll()
        log("Results: $results")
    }
}
```
- **설명:**
  - 여러 async 코루틴의 결과를 awaitAll()로 한 번에 기다립니다.
  - 모든 코루틴이 완료되면 결과 리스트를 반환합니다.
  - **의도:** 다중 async 작업의 일괄 처리 패턴을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **async 사용 시나리오:**
  - 독립적인 작업들의 병렬 실행
  - 결과 값이 필요한 비동기 작업
  - 조건부 실행과 지연 시작
- **참고 자료:**
  - [Kotlin 공식 문서: Async](https://kotlinlang.org/docs/coroutines-basics.html#concurrent-using-async) 