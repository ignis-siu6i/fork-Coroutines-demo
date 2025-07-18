# ScopedActivity

## 1. lifecycleScope와 supervisorScope, 예외 처리

```kotlin
lifecycleScope.launch {
    supervisorScope {
        launch {
            // child 1: 예외 발생
            throw RuntimeException("OOPS!")
        }
        launch {
            // child 2: 정상 동작
        }
    }
}
```
- **설명:**
  supervisorScope를 사용하면 자식 코루틴 중 하나가 예외로 종료되어도 다른 자식은 영향을 받지 않습니다. lifecycleScope와 결합해 생명주기 안전성과 에러 격리를 동시에 구현할 수 있습니다.

---

## 2. CoroutineExceptionHandler로 예외 잡기

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e(TAG, "CoroutineExceptionHandler got $exception")
}
lifecycleScope.launch(handler) {
    // ...
}
```
- **설명:**
  CoroutineExceptionHandler를 사용하면 코루틴 내에서 발생한 예외를 안전하게 처리할 수 있습니다. 로그 출력, 사용자 알림 등 다양한 후처리가 가능합니다.

---

## 3. 결론 및 참고

- **Android에서 supervisorScope, 예외 처리의 활용:**
  - 자식 코루틴의 독립적 실패 처리, 생명주기 안전, 에러 로깅 등

- **참고 자료:**
  - [공식 문서: supervisorScope](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html)
  - [공식 문서: CoroutineExceptionHandler](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/) 