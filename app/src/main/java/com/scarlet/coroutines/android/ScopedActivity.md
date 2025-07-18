# ScopedActivity

이 파일은 Android 환경에서 supervisorScope를 사용한 예외 처리를 설명합니다. 자식 코루틴 중 하나가 실패해도 다른 자식에게 영향을 주지 않는 방법과 CoroutineExceptionHandler의 설치 위치에 따른 동작 차이를 보여줍니다.

---

## onCreate - supervisorScope를 사용한 예외 격리
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    val handler = CoroutineExceptionHandler { _, exception ->
        Log.e(TAG, "CoroutineExceptionHandler got $exception")
    }

    lifecycleScope.launch {
        Log.i(TAG, "parent started")

        supervisorScope {
            coroutineContext.job.invokeOnCompletion { ex ->
                Log.e(TAG, "supervisorScope: isCancelled = ${coroutineContext.job.isCancelled}, cause = $ex")
            }

            // TODO: Install the custom handler here ...
            launch {
                Log.e(TAG, "child 1 started")
                delay(2_000)
                Log.e(TAG, "child 1: I'm about to throwing exception")
                throw RuntimeException("OOPS!")
            }.apply {
                invokeOnCompletion {
                    Log.e(TAG, "Child 1: isCancelled = $isCancelled, cause = $it")
                }
            }

            launch {
                Log.e(TAG, "child 2 started")
                delay(5_000)
            }.apply {
                invokeOnCompletion {
                    Log.e(TAG, "Child 2: isCancelled = $isCancelled, cause = $it")
                }
            }
        }

    }.apply {
        invokeOnCompletion {
            Log.e(TAG, "Parent: isCancelled = $isCancelled, cause = $it")
        }
    }
}
```
- **설명:**
  - `supervisorScope` 내에서 두 개의 자식 코루틴을 시작합니다.
  - 첫 번째 자식은 2초 후 RuntimeException을 던집니다.
  - 두 번째 자식은 5초 동안 실행됩니다.
  - **의도:** supervisorScope가 자식 간의 예외 격리를 어떻게 제공하는지 실험적으로 보여줍니다.

---

## CoroutineExceptionHandler 설치 위치 실험
```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e(TAG, "CoroutineExceptionHandler got $exception")
}

// TODO: Install the custom handler here ...
launch {
    // 여기에 handler를 설치하면?
}
```
- **설명:**
  - CoroutineExceptionHandler를 정의하지만 아직 설치하지 않았습니다.
  - TODO 주석은 handler를 어디에 설치해야 하는지 실험해보라는 의미입니다.
  - **의도:** CoroutineExceptionHandler의 올바른 설치 위치와 동작 범위를 실험적으로 보여줍니다.

---

## invokeOnCompletion을 통한 상태 추적
```kotlin
coroutineContext.job.invokeOnCompletion { ex ->
    Log.e(TAG, "supervisorScope: isCancelled = ${coroutineContext.job.isCancelled}, cause = $ex")
}

launch {
    // ...
}.apply {
    invokeOnCompletion {
        Log.e(TAG, "Child 1: isCancelled = $isCancelled, cause = $it")
    }
}
```
- **설명:**
  - 각 Job의 완료 상태와 취소 원인을 추적합니다.
  - supervisorScope와 각 자식 코루틴의 상태를 개별적으로 모니터링합니다.
  - **의도:** 예외 발생 시 각 코루틴의 상태 변화를 세밀하게 관찰할 수 있게 해줍니다.

---

## 생명주기 콜백 모니터링
```kotlin
override fun onStart() {
    super.onStart()
    Log.e(TAG, "[onStart]")
}

override fun onDestroy() {
    super.onDestroy()
    Log.e(TAG, "[onDestroy]")
}
```
- **설명:**
  - 액티비티 생명주기와 코루틴 실행의 관계를 추적합니다.
  - 예외 발생 시 액티비티 상태에 미치는 영향을 관찰할 수 있습니다.
  - **의도:** Android 환경에서 코루틴 예외가 UI에 미치는 영향을 실험적으로 보여줍니다.

---

## 실험 시나리오
1. **기본 실행**: 첫 번째 자식이 예외를 던질 때 두 번째 자식이 계속 실행되는지 확인
2. **ExceptionHandler 설치**: TODO 부분에 handler를 다양한 위치에 설치하며 동작 차이 확인
   - `lifecycleScope.launch(handler)`
   - `launch(handler)` (자식 코루틴에)
   - `supervisorScope` 내부
3. **로그 분석**: 각 코루틴의 isCancelled 상태와 완료 원인 분석

---

## ExceptionHandler 설치 옵션
```kotlin
// 옵션 1: 부모 코루틴에 설치
lifecycleScope.launch(handler) {
    supervisorScope { ... }
}

// 옵션 2: 자식 코루틴에 설치
launch(handler) {
    throw RuntimeException("OOPS!")
}

// 옵션 3: supervisorScope 매개변수로 전달 (실제로는 불가능)
```

---

## 결론 및 참고
- **Android에서 supervisorScope 활용:**
  - 독립적인 자식 작업들의 격리
  - 일부 실패가 전체에 영향을 주지 않음
  - 적절한 ExceptionHandler 설치 위치 선택
  - 생명주기와 연결된 안전한 예외 처리
- **실험을 통한 학습:**
  - handler 설치 위치 변경하며 동작 확인
  - 로그를 통한 상태 추적과 분석
  - Android 환경에서의 예외 처리 패턴 이해
- **참고 자료:**
  - [Kotlin 공식 문서: Exception Handling](https://kotlinlang.org/docs/exception-handling.html)
  - [Android 코루틴 예외 처리](https://developer.android.com/kotlin/coroutines/coroutines-best-practices#exceptions) 