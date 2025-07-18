# C03_CooperationForCancellation

이 파일은 협조적 취소(Cooperative Cancellation)의 중요성을 설명합니다. CPU 집약적인 작업에서 취소에 협조하지 않는 코루틴의 문제점과 isActive, ensureActive(), yield() 등을 사용한 해결책을 보여줍니다.

---

## UnCooperative_vs_Cooperative_Cancellation.main
```kotlin
object UnCooperative_vs_Cooperative_Cancellation {
    // How to make sure this suspending function be cooperative?
    private suspend fun printTwice() = withContext(Dispatchers.Default) {
        val startTime = System.currentTimeMillis()
        var nextPrintTime = startTime
        while (true) {
            if (System.currentTimeMillis() >= nextPrintTime) {
                log("I'm working..")
                nextPrintTime += 500
            }
        }
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            printTwice()
        }

        delay(1_500)
        log("Cancelling job ...")
        job.cancelAndJoin()
    }
}
```
- **설명:**
  - 무한 루프에서 suspend 함수 호출 없이 CPU 작업만 수행하는 비협조적 코루틴입니다.
  - `while (true)` 루프에 suspend point가 없어서 취소 요청을 확인할 기회가 없습니다.
  - job.cancelAndJoin()을 호출해도 코루틴이 계속 실행되어 프로그램이 종료되지 않습니다.
  - **의도:** 협조적 취소가 없을 때의 문제점을 실험적으로 보여줍니다.

---

## Cleanup_When_Cancelled.main
```kotlin
object Cleanup_When_Cancelled {
    private suspend fun printTwice() = withContext(Dispatchers.Default) {
        val startTime = System.currentTimeMillis()
        var nextPrintTime = startTime
        while (isActive) {
            if (System.currentTimeMillis() >= nextPrintTime) {
                log("job: I'm working..")
                nextPrintTime += 500
            }
        }

        // TODO: cleanup
        log("job: I'm cancelled")
        cleanUp()
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            printTwice()
        }

        delay(1_500)

        log("Try to cancel the job ...")
        job.cancelAndJoin()
    }

    private suspend fun cleanUp() {
        delay(100)
        log("Cleanup ...")
    }
}
```
- **설명:**
  - `while (isActive)` 조건을 사용하여 취소 상태를 주기적으로 확인합니다.
  - 코루틴이 취소되면 `isActive`가 false가 되어 루프를 종료합니다.
  - 루프 종료 후 정리 작업(cleanUp())을 수행할 수 있습니다.
  - **의도:** 협조적 취소의 올바른 구현과 정리 작업 패턴을 실험적으로 보여줍니다.

---

## 협조적 취소를 위한 방법들

### 1. isActive 확인
```kotlin
while (isActive) {
    // CPU 집약적 작업
}
```

### 2. ensureActive() 호출
```kotlin
while (true) {
    ensureActive() // CancellationException을 던짐
    // CPU 집약적 작업
}
```

### 3. yield() 호출
```kotlin
while (true) {
    yield() // 다른 코루틴에게 실행 기회 제공 + 취소 확인
    // CPU 집약적 작업
}
```

### 4. 주기적인 delay() 호출
```kotlin
while (true) {
    // CPU 집약적 작업
    delay(1) // 아주 짧은 지연으로 취소 확인
}
```

---

## 실험 시나리오
1. **비협조적 취소 테스트**: 
   - 첫 번째 main 함수 실행
   - 1.5초 후 취소 시도하지만 계속 실행되는 것 확인
   
2. **협조적 취소 테스트**:
   - 두 번째 main 함수 실행
   - 1.5초 후 취소가 정상적으로 동작하는 것 확인
   
3. **다양한 협조 방법 실험**:
   - `while (isActive)` → `while (true) { ensureActive(); ... }`
   - `while (isActive)` → `while (true) { yield(); ... }`
   - 각각의 동작 차이 확인

---

## 정리 작업 주의사항
```kotlin
// 정리 작업에서는 NonCancellable 사용을 고려
private suspend fun cleanUp() {
    withContext(NonCancellable) {
        delay(100) // 취소되지 않는 정리 작업
        log("Cleanup completed")
    }
}
```

---

## 결론 및 참고
- **협조적 취소의 핵심 원칙:**
  - CPU 집약적 작업에서 주기적인 취소 상태 확인
  - 적절한 suspend point 제공
  - 정리 작업을 위한 안전한 종료 로직
  - 사용자 경험을 위한 반응성 확보
- **실용적 패턴:**
  - 긴 반복 작업에서 isActive 확인
  - 계산 작업 중간에 yield() 호출
  - 정리 작업에서 NonCancellable 활용
- **참고 자료:**
  - [Kotlin 공식 문서: Making computation code cancellable](https://kotlinlang.org/docs/cancellation-and-timeouts.html#making-computation-code-cancellable)
  - [Cooperative cancellation](https://elizarov.medium.com/cooperative-cancellation-65b9e5113f00) 