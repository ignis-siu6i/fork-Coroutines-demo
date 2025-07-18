# C02_NonCancellable

이 파일은 코루틴이 취소 상태(Cancelling)일 때의 특별한 동작을 설명합니다. 취소 중인 코루틴에서 새로운 코루틴 시작이나 suspend 함수 호출이 어떻게 처리되는지, 그리고 정리 작업을 위한 NonCancellable 컨텍스트 사용법을 보여줍니다.

---

## Launch_in_Canceling_State_Will_Be_Ignored.main
```kotlin
object Launch_in_Canceling_State_Will_Be_Ignored {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            try {
                delay(200)
                log("Unreachable code") // because it will be cancelled after 100ms
            } finally {
                log("Finally")

                log("isActive = ${coroutineContext.isActive}, isCancelled = ${coroutineContext.job.isCancelled}")

                // Try to launch new coroutine in cancelling state
                launch { // will be ignored because of immediate cancellation
                    log("Will not be printed")
                    delay(50)
                }.onCompletion("Jombi")
                // .join() // will throw cancellation exception and skip the rest

                log("Check whether control flow can reach here ...")
            }
        }

        delay(100)
        job.cancelAndJoin()
        log("Cancel done")
    }
}
```
- **설명:**
  - 코루틴이 취소 상태에 있을 때 새로운 자식 코루틴을 시작하려고 하면 무시됩니다.
  - `isActive = false`, `isCancelled = true` 상태에서 launch는 즉시 취소됩니다.
  - join()을 호출하면 CancellationException이 발생하여 나머지 코드가 실행되지 않습니다.
  - **의도:** 취소 상태에서 새로운 코루틴 생성의 제약과 동작을 실험적으로 보여줍니다.

---

## Call_Suspending_Function_in_Canceling_State_Will_Throw_Cancellation_Exception.main
```kotlin
object Call_Suspending_Function_in_Canceling_State_Will_Throw_Cancellation_Exception {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            try {
                delay(200)
                log("Unreachable code") // because it will be cancelled after 100ms
            } finally {
                log("Finally")

                log("isActive = ${coroutineContext.isActive}, isCancelled = ${coroutineContext.job.isCancelled}")

                // Try to call suspending function will throw cancellation exception
                try {
                    delay(100)
                    log("Will not be printed")
                } catch (ex: Exception) {
                    log("Caught: $ex")
                }
                // Nevertheless, if you want to call suspending function to clean up ... how to do?
            }
        }

        delay(100)
        job.cancelAndJoin()
        log("Cancel done")
    }
}
```
- **설명:**
  - 취소 상태에서 suspend 함수(delay 등)를 호출하면 CancellationException이 발생합니다.
  - try-catch로 예외를 잡을 수 있지만, 정리 작업을 위한 suspend 함수 호출은 여전히 문제가 됩니다.
  - **의도:** 취소 상태에서 suspend 함수 호출의 제약과 정리 작업의 어려움을 실험적으로 보여줍니다.

---

## Call_Suspending_Function_in_Cancelling_State_To_Cleanup.main
```kotlin
object Call_Suspending_Function_in_Cancelling_State_To_Cleanup {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            try {
                delay(200)
                log("Coroutine finished")
            } finally {
                log("Finally")

                // Use `withContext(NonCancellable)`.
                // DO NOT USE `NonCancellable` with `launch` or `async`
                cleanUp()
            }
        }.onCompletion("Job")

        delay(100)
        job.cancelAndJoin()
        log("Cancel done")
    }

    private suspend fun cleanUp() {
        log("Cleaning up starts ...")
        delay(2_000)
        log("Cleaning up ...done")
    }
}
```
- **설명:**
  - `withContext(NonCancellable)`을 사용하여 취소 상태에서도 정리 작업을 수행할 수 있습니다.
  - NonCancellable은 launch나 async와 함께 사용하면 안 되고, withContext와만 사용해야 합니다.
  - 정리 작업은 취소되지 않고 완료될 때까지 실행됩니다.
  - **의도:** NonCancellable 컨텍스트를 사용한 안전한 정리 작업 패턴을 실험적으로 보여줍니다.

---

## 취소 상태에서의 제약사항
1. **새로운 자식 코루틴**: 즉시 취소되어 무시됨
2. **suspend 함수 호출**: CancellationException 발생
3. **join() 호출**: CancellationException 발생하여 후속 코드 실행 중단

---

## NonCancellable 사용 지침
```kotlin
// ✅ 올바른 사용법
withContext(NonCancellable) {
    // 정리 작업
    delay(1000) // 취소되지 않음
    cleanupResources()
}

// ❌ 잘못된 사용법
launch(NonCancellable) {
    // 권장되지 않음
}
```

---

## 실험 시나리오
1. **첫 번째 실험**: launch 무시 확인 및 join() 주석 해제 시 동작 차이
2. **두 번째 실험**: finally 블록에서 다양한 suspend 함수 호출 시도
3. **세 번째 실험**: cleanUp() 함수에 withContext(NonCancellable) 추가/제거 비교

---

## 결론 및 참고
- **취소 상태에서의 동작 원칙:**
  - 새로운 작업 시작 금지
  - suspend 함수 호출 제한
  - 정리 작업을 위한 NonCancellable 활용
  - 안전한 리소스 정리 보장
- **실용적 패턴:**
  - finally 블록에서 withContext(NonCancellable) 사용
  - 적절한 예외 처리와 정리 로직 분리
  - 취소 상태 확인을 통한 조건부 실행
- **참고 자료:**
  - [Kotlin 공식 문서: NonCancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable/)
  - [Cancellation and timeouts](https://kotlinlang.org/docs/cancellation-and-timeouts.html) 