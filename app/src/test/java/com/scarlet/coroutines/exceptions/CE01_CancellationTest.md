# CE01_CancellationTest

이 파일은 코루틴 취소 동작을 테스트하는 다양한 시나리오를 설명합니다. 협조적 취소 vs 비협조적 취소, CancellationException 처리의 올바른 방법과 잘못된 방법을 실제 테스트 코드로 보여줍니다.

---

## 테스트용 함수들
```kotlin
private suspend fun networkRequestCooperative() {
    log("networkRequestCooperative (🫱🏼)")
    delay(3_000)
    log("networkRequestCooperative done (🫱🏼)")
}

private suspend fun networkRequestFailed() {
    log("networkRequestFailed (❌)")
    delay(1_000)
    log("throwing exception from networkRequestFailed (❌)")
    throw RuntimeException("Oops...from networkRequestFailed (❌)")
}

private suspend fun networkRequestUncooperative() = coroutineScope {
    fun fib(n: Long): Long = if (n <= 1) n else fib(n - 1) + fib(n - 2)

    log("networkRequestUncooperative (🔒)")
    log("I AM BUSY ... NO INTERRUPTIONS PLEASE ...")
    log("fib(45) = ${fib(45)}") // simulating blocking operation
    log("networkRequestUncooperative done (🔒)")
}
```

### 함수별 특징
- **networkRequestCooperative**: delay()를 사용하는 협조적 함수
- **networkRequestFailed**: 의도적으로 예외를 발생시키는 함수
- **networkRequestUncooperative**: CPU 집약적 작업으로 취소에 비협조적인 함수

---

## 비협조적 코루틴 취소 테스트
```kotlin
@Test
fun `uncooperative coroutine cannot be cancelled`() = runTest {
    val job = launch {
        networkRequestUncooperative()

        log("Am I printed?, first check")

        delay(100)

        log("Am I printed?, second check")
    }.onCompletion("job")

    delay(100)
//    job.cancelAndJoin()
}
```

### 테스트 특징
- **문제점**: `fib(45)` 계산 중에는 suspend point가 없어 취소 불가능
- **결과**: 취소 요청이 무시되고 계속 실행됨
- **주석 해제**: `job.cancelAndJoin()` 주석을 해제하여 취소 시도 가능
- **의도**: 비협조적 코루틴의 문제점을 실험적으로 보여줍니다.

---

## 협조적 코루틴 취소 테스트
```kotlin
@Test
fun `cooperative coroutine can be canceled`() = runBlocking {
    val job = launch {
        networkRequestCooperative()

        log("Am I printed?, first check")

        delay(100)

        log("Am I printed?, second check")
    }.onCompletion("Job")

    delay(100)
    job.cancelAndJoin()
}
```

### 테스트 특징
- **성공적 취소**: `delay(3_000)` 중에 취소 요청 감지
- **결과**: 함수가 중단되고 후속 코드도 실행되지 않음
- **runBlocking 사용**: 실제 취소 동작을 관찰하기 위함
- **의도**: 협조적 취소의 올바른 동작을 실험적으로 보여줍니다.

---

## 실패한 함수의 예외 처리
```kotlin
@Test
fun `failed suspending function throws causing exception, not cancellation exception`() =
    runTest {
        launch {
            try {
                networkRequestFailed()
            } catch (ex: Exception) {
                log("Caught exception = $ex")
            }
        }
    }
```

### 테스트 특징
- **예외 구분**: 일반 예외 vs CancellationException 구분
- **정상 예외 처리**: RuntimeException은 정상적으로 catch됨
- **결과**: "Caught exception = java.lang.RuntimeException: Oops..." 출력
- **의도**: 일반 예외와 취소 예외의 차이를 실험적으로 보여줍니다.

---

## CancellationException을 삼키는 잘못된 예제
```kotlin
@Test
fun `cancellation exception swallowed - so, next suspend function starts running`() = runTest {
    val job = launch {
        log("Coroutine starts running ... isActive = $isActive")

        try {
            networkRequestCooperative()
        } catch (ex: Exception) { // CATCH ALL EXCEPTIONS including CancellationException
            log("Caught: ${ex.javaClass.simpleName}")
        }

        log("Coroutine keep running ... isActive = $isActive")
        networkRequestUncooperative() // this will not be skipped
    }.onCompletion("job")

    delay(100)
    job.cancelAndJoin()
}
```

### 잘못된 패턴의 문제점
- **CancellationException 포획**: 취소 신호를 삼켜버림
- **계속 실행**: isActive = false임에도 코루틴 계속 실행
- **위험한 동작**: `networkRequestUncooperative()`까지 실행됨
- **결과**: 취소가 의도대로 동작하지 않음
- **의도**: CancellationException을 잡으면 안 되는 이유를 실험적으로 보여줍니다.

---

## CancellationException 올바른 처리
```kotlin
@Test
fun `cancellation caught, but rethrown - remaining computation all skipped`() = runTest {
    val job = launch {
        try {
            networkRequestCooperative()
        } catch (ex: Exception) {
            log("Caught: ${ex.javaClass.simpleName}")
            if (ex is CancellationException) {
                throw ex
            }
        }

        log("All subsequent computations will be skipped ...")
        networkRequestUncooperative() // long running computation
        log("This will not be printed")

    }.onCompletion("job")

    delay(100)

    job.cancelAndJoin()
}
```

### 올바른 패턴의 특징
- **재던지기**: CancellationException을 다시 던짐
- **즉시 중단**: 후속 계산이 모두 건너뛰어짐
- **안전한 취소**: 취소 신호가 올바르게 전파됨
- **결과**: "All subsequent computations will be skipped ..." 출력되지 않음
- **의도**: CancellationException의 올바른 처리 방법을 실험적으로 보여줍니다.

---

## CancellationException 처리 규칙

### ❌ 잘못된 방법
```kotlin
try {
    // suspend 함수 호출
} catch (ex: Exception) {
    // CancellationException도 함께 catch - 위험!
    log("Caught: $ex")
}
```

### ✅ 올바른 방법 1 - 재던지기
```kotlin
try {
    // suspend 함수 호출
} catch (ex: Exception) {
    log("Caught: $ex")
    if (ex is CancellationException) {
        throw ex  // 반드시 재던지기
    }
}
```

### ✅ 올바른 방법 2 - 구체적 예외만 catch
```kotlin
try {
    // suspend 함수 호출
} catch (ex: IOException) {
    // 구체적인 예외만 catch
    log("Network error: $ex")
} catch (ex: IllegalArgumentException) {
    // 또 다른 구체적 예외
    log("Invalid argument: $ex")
}
```

---

## 실험 시나리오

### 1. 비협조적 코루틴 실험
1. **주석 해제**: `job.cancelAndJoin()` 주석 해제
2. **실행 관찰**: 취소 요청이 무시되는지 확인
3. **로그 분석**: "Am I printed?" 메시지들이 출력되는지 확인

### 2. CancellationException 처리 실험
1. **잘못된 예제**: catch 블록에서 CancellationException 삼키기
2. **올바른 예제**: CancellationException 재던지기
3. **결과 비교**: 후속 코드 실행 여부 확인

### 3. 다양한 예외 시나리오
1. **일반 예외**: RuntimeException 처리
2. **취소 예외**: CancellationException 처리
3. **혼합 시나리오**: 여러 예외가 섞인 경우

---

## 결론 및 참고
- **취소 처리 원칙:**
  - CancellationException은 절대 삼키지 말 것
  - 협조적 취소를 위해 적절한 suspend point 제공
  - 구체적인 예외 타입만 catch하거나 CancellationException 재던지기
- **테스트 작성 시 주의사항:**
  - 취소 동작을 정확히 테스트하기 위해 적절한 delay 사용
  - runTest vs runBlocking 선택에 따른 동작 차이 고려
  - 예외 처리가 올바른지 확인하는 테스트 작성
- **참고 자료:**
  - [Kotlin 공식 문서: Cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
  - [CancellationException handling](https://elizarov.medium.com/how-to-handle-exceptions-in-kotlin-coroutines-667b6e1b3b3c) 