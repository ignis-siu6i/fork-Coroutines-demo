# A04_ExceptionHandling

이 파일은 코루틴에서의 예외 처리 메커니즘을 설명합니다. launch vs async의 예외 처리 차이, CoroutineExceptionHandler 사용법, SupervisorJob과 supervisorScope의 예외 격리, 예외 전파 규칙을 다양한 예제로 보여줍니다.

---

## Launch_Exception_Demo.main
```kotlin
object Launch_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            log("Throwing exception from launch")
            throw IndexOutOfBoundsException() // Will be printed to console
        }
        job.join()
        log("Joined failed job")
    }
}
```
- **설명:**
  - launch에서 발생한 예외는 즉시 부모 코루틴으로 전파되어 취소를 일으킵니다.
  - try-catch로 감싸도 예외를 잡을 수 없습니다.
  - **의도:** launch의 "fire-and-forget" 특성과 예외 즉시 전파를 실험적으로 보여줍니다.

---

## Async_Exception_Demo.main
```kotlin
object Async_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferred = async {
            log("Throwing exception from async")
            throw ArithmeticException() // Nothing is printed, relying on user to call await
        }
        try {
            deferred.await()
        } catch (e: ArithmeticException) {
            log("Caught ArithmeticException")
        }
    }
}
```
- **설명:**
  - async에서 발생한 예외는 await() 호출 시점까지 지연됩니다.
  - await()를 호출하지 않으면 예외가 무시될 수 있습니다.
  - **의도:** async의 지연된 예외 처리와 명시적 처리 필요성을 실험적으로 보여줍니다.

---

## ExceptionHandler_Demo.main
```kotlin
object ExceptionHandler_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val handler = CoroutineExceptionHandler { _, exception ->
            log("Caught $exception")
        }
        
        val job = launch(handler) {
            throw AssertionError()
        }
        job.join()
    }
}
```
- **설명:**
  - CoroutineExceptionHandler로 처리되지 않은 예외를 전역적으로 처리할 수 있습니다.
  - root coroutine(launch, async)에서만 동작합니다.
  - **의도:** 전역 예외 처리 메커니즘과 적용 범위를 실험적으로 보여줍니다.

---

## SupervisorJob_Exception_Demo.main
```kotlin
object SupervisorJob_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val supervisor = SupervisorJob()
        with(CoroutineScope(coroutineContext + supervisor)) {
            val child1 = launch {
                delay(100)
                log("Child 1 throws exception")
                throw AssertionError()
            }
            val child2 = launch {
                delay(200)
                log("Child 2 completes normally")
            }
            joinAll(child1, child2)
        }
    }
}
```
- **설명:**
  - SupervisorJob은 한 자식의 실패가 다른 자식에게 전파되지 않도록 격리합니다.
  - 각 자식은 독립적으로 실패할 수 있습니다.
  - **의도:** SupervisorJob의 예외 격리 메커니즘을 실험적으로 보여줍니다.

---

## SupervisorScope_Exception_Demo.main
```kotlin
object SupervisorScope_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        supervisorScope {
            val child1 = launch {
                delay(100)
                log("Child 1 throws exception")
                throw AssertionError()
            }
            val child2 = launch {
                delay(200)
                log("Child 2 completes normally")
            }
        }
    }
}
```
- **설명:**
  - supervisorScope는 임시적인 supervisor job을 만들어 예외 격리를 제공합니다.
  - SupervisorJob을 직접 사용하는 것보다 간편한 방법입니다.
  - **의도:** supervisorScope의 편리한 예외 격리 패턴을 실험적으로 보여줍니다.

---

## Nested_Exception_Demo.main
```kotlin
object Nested_Exception_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val handler = CoroutineExceptionHandler { _, exception ->
            log("Caught $exception")
        }
        
        val outer = launch(handler) {
            launch { // 중첩된 코루틴
                throw IOException()
            }
        }
        outer.join()
    }
}
```
- **설명:**
  - 중첩된 코루틴에서 발생한 예외도 root 코루틴의 ExceptionHandler에서 처리됩니다.
  - 예외는 부모로 전파되어 최상위에서 처리됩니다.
  - **의도:** 중첩된 코루틴의 예외 전파 경로를 실험적으로 보여줍니다.

---

## 결론 및 참고
- **예외 처리 원칙:**
  - launch: 즉시 전파, CoroutineExceptionHandler 사용
  - async: await() 시점 전파, try-catch 사용
  - SupervisorJob/supervisorScope: 예외 격리
- **참고 자료:**
  - [Kotlin 공식 문서: Exception Handling](https://kotlinlang.org/docs/exception-handling.html) 