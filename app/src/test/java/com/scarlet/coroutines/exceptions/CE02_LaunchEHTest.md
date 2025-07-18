# CE02_LaunchEHTest

이 파일은 `launch`로 실행된 코루틴에서 발생하는 예외 처리 메커니즘을 설명합니다. Root 코루틴의 예외 처리, `runBlocking` vs `runTest`의 차이점, 그리고 예외 전파 방식을 실제 테스트 시나리오로 학습할 수 있습니다.

---

## 기본 예외 처리 함수
```kotlin
private fun failingFunction() {
    throw RuntimeException("Oops(❌)")
}
```

### 용도
- 의도적인 예외 발생으로 예외 처리 메커니즘 테스트
- 다양한 코루틴 컨텍스트에서의 예외 동작 확인

---

## 직접 예외 발생 테스트
```kotlin
@Test(expected = RuntimeException::class)
fun `exception thrown`() {
    failingFunction()
}
```

### 기본 동작
- **일반 함수**: 예외가 즉시 호출자에게 전파
- **예상 동작**: RuntimeException이 테스트에서 catch됨
- **의도**: 기본적인 예외 전파 메커니즘 확인

---

## runBlocking/runTest에서의 예외 재던지기
```kotlin
@Test(expected = RuntimeException::class)
fun `exception with runBlocking or runTest`() = runTest {
    failingFunction()
}
```

### 특징
- **재던지기**: `runBlocking`과 `runTest` 모두 uncaught exception을 재던짐
- **동일한 동작**: 두 함수 모두 동일한 예외 처리 방식
- **의도**: 테스트 환경에서의 기본 예외 처리 확인

---

## 첫 번째 예외만 재던지기
```kotlin
@Test
fun `rethrows only the first uncaught exception`() = runTest {
    onCompletion("runTest")

    launch {
        delay(10)
        throw RuntimeException("Yellow(🌕)")
    }
    launch {
        delay(50)
        throw IOException("Mellow(🍈)")
    }
}
```

### 예외 우선순위
- **첫 번째 예외**: 더 빨리 발생한 RuntimeException만 재던져짐
- **후속 예외**: IOException은 무시됨
- **시간 순서**: delay(10) vs delay(50)으로 순서 결정
- **의도**: 여러 예외 발생 시 첫 번째 예외만 처리됨을 보여줌

---

## 현장에서 예외 처리 (성공)
```kotlin
@Test
fun `can handle rethrown exception on site using try-catch`() = runTest {
    launch {
        try {
            failingFunction()
        } catch (ex: Exception) {
            log("Caught $ex")
        }
    }
}
```

### 성공 패턴
- **즉시 처리**: 예외 발생 지점에서 직접 처리
- **전파 방지**: 예외가 상위로 전파되지 않음
- **정상 완료**: 테스트가 성공적으로 완료
- **의도**: 적절한 예외 처리 방법 시연

---

## 현장에서 예외 처리 (실패)
```kotlin
@Test(expected = RuntimeException::class)
fun `propagated exception from nested coroutines cannot be handled on site using try-catch`() =
    runTest {
        try {
            launch {
                failingFunction()
            }
        } catch (ex: Exception) {
            log("Caught $ex")  // useless
        }
    }
```

### 실패 패턴
- **잘못된 위치**: launch 외부에서 try-catch 시도
- **전파 불가**: launch 내부 예외는 외부 try-catch로 잡히지 않음
- **예외 재던지기**: 결국 runTest에서 예외가 재던져짐
- **의도**: 잘못된 예외 처리 방법의 문제점 시연

---

## runBlocking vs runTest 차이점

### runBlocking의 동작
```kotlin
@Test // Try runBlocking ...
fun `Failure of child cancels the parent and its siblings1`() = runTest {
    onCompletion("runTest")

    // Same behavior even if `SupervisorJob` is used.
    val scope = CoroutineScope(Job()).onCompletion("scope")

    val parentJob = scope.launch {
        launch {
            delay(100)
            throw RuntimeException("oops(❌)")
        }.onCompletion("child1")

        launch {
            delay(1_000)
        }.onCompletion("child2")
    }.onCompletion("parentJob")

    parentJob.join()

    scope.completeStatus("scope")
}
```

### 핵심 차이점
- **runBlocking**: Scope에 전파된 uncaught exception을 재던지지 않음
- **runTest**: 모든 uncaught exception을 재던짐
- **SupervisorJob**: Job 타입과 관계없이 동일한 동작
- **의도**: 두 테스트 함수의 예외 처리 차이점 비교

---

## 자식 실패로 인한 부모/형제 취소
```kotlin
@Test // Try runBlocking ...
fun `Failure of child cancels the parent and its siblings2`() = runTest {
    onCompletion("runTest")

    val parentJob = launch {
        launch {
            delay(100)
            throw RuntimeException("oops(❌)")
        }.onCompletion("child1")

        launch {
            delay(1_000)
        }.onCompletion("child2")
    }.onCompletion("parentJob")

    parentJob.join()
}
```

### 취소 전파 메커니즘
- **자식 실패**: child1에서 예외 발생
- **부모 취소**: parentJob이 취소됨
- **형제 취소**: child2도 함께 취소됨
- **예외 재던지기**: runTest에서 첫 번째 예외 재던지기
- **의도**: 구조화된 동시성의 실패 전파 방식 시연

---

## 실제 예외 처리 패턴

### 1. 개별 코루틴 예외 처리
```kotlin
launch {
    try {
        riskyOperation()
    } catch (e: SpecificException) {
        handleSpecificError(e)
    } catch (e: Exception) {
        handleGenericError(e)
    }
}
```

### 2. CoroutineExceptionHandler 사용
```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, exception ->
    log("Caught $exception")
}

val scope = CoroutineScope(Job() + exceptionHandler)
scope.launch {
    failingFunction() // 예외가 handler로 전달됨
}
```

### 3. SupervisorJob을 사용한 격리
```kotlin
val supervisorScope = CoroutineScope(SupervisorJob())

supervisorScope.launch {
    // 이 코루틴의 실패가 다른 코루틴에 영향 주지 않음
    failingFunction()
}

supervisorScope.launch {
    // 독립적으로 실행됨
    normalOperation()
}
```

---

## 실험 시나리오

### 1. runBlocking vs runTest 비교
1. **runTest → runBlocking 변경**: 동일한 테스트를 runBlocking으로 실행
2. **예외 동작 관찰**: 어떤 예외가 재던져지는지 확인
3. **로그 분석**: 각각의 완료 상태 비교

### 2. 예외 처리 위치 실험
1. **올바른 위치**: launch 내부에서 try-catch
2. **잘못된 위치**: launch 외부에서 try-catch
3. **결과 비교**: 어떤 방법이 효과적인지 확인

### 3. 구조화된 동시성 실험
1. **자식 실패**: 한 자식 코루틴에서 예외 발생
2. **전파 관찰**: 부모와 형제 코루틴의 취소 확인
3. **onCompletion 로그**: 각 코루틴의 완료 상태 추적

---

## 모범 사례

### ✅ 권장: 적절한 예외 처리 위치
```kotlin
launch {
    try {
        riskyOperation()
    } catch (e: Exception) {
        handleError(e) // 예외 발생 지점에서 처리
    }
}
```

### ✅ 권장: CoroutineExceptionHandler 사용
```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    logError(exception)
}
val scope = CoroutineScope(SupervisorJob() + handler)
```

### ✅ 권장: 구체적인 예외 타입 처리
```kotlin
try {
    networkCall()
} catch (e: IOException) {
    handleNetworkError(e)
} catch (e: JsonException) {
    handleParsingError(e)
}
```

### ❌ 피해야 할 패턴: 외부 try-catch
```kotlin
try {
    launch {
        failingFunction() // 이 예외는 외부 catch로 잡히지 않음
    }
} catch (e: Exception) {
    // 실행되지 않음
}
```

### ❌ 피해야 할 패턴: 모든 예외 무시
```kotlin
launch {
    try {
        riskyOperation()
    } catch (e: Exception) {
        // 예외 무시 - 디버깅 어려움
    }
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - launch로 시작된 코루틴의 예외는 상위로 전파됨
  - 예외 처리는 발생 지점에서 즉시 해야 함
  - runTest와 runBlocking의 예외 재던지기 차이점
- **구조화된 동시성:**
  - 자식 코루틴의 실패는 부모와 형제에게 전파됨
  - SupervisorJob으로 실패 격리 가능
  - CoroutineExceptionHandler로 전역 예외 처리
- **참고 자료:**
  - [Exception handling in coroutines](https://kotlinlang.org/docs/exception-handling.html)
  - [Structured concurrency](https://elizarov.medium.com/structured-concurrency-722d765aa952) 