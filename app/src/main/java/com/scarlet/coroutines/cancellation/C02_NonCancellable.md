# C02_NonCancellable

## 1. 취소 중 launch/async/suspend 호출 시 동작

```kotlin
val job = launch {
    try {
        delay(200)
    } finally {
        launch { /* 취소 중 launch는 무시됨 */ }
    }
}
job.cancelAndJoin()
```
- **설명:**
  코루틴이 취소 상태일 때 launch/async로 새로운 코루틴을 시작하면 즉시 무시됩니다. finally 블록 내에서 launch를 호출해도 실행되지 않습니다.

---

## 2. 취소 중 suspend 함수 호출 시 예외

```kotlin
val job = launch {
    try {
        delay(200)
    } finally {
        delay(100) // CancellationException 발생
    }
}
job.cancelAndJoin()
```
- **설명:**
  취소 중에 suspend 함수를 호출하면 CancellationException이 발생합니다. cleanup 등에서 suspend 함수가 필요하다면 NonCancellable 컨텍스트를 사용해야 합니다.

---

## 3. NonCancellable 컨텍스트로 cleanup

```kotlin
private suspend fun cleanUp() {
    withContext(NonCancellable) {
        delay(2_000)
        log("Cleaning up ...done")
    }
}
```
- **설명:**
  NonCancellable 컨텍스트를 사용하면 코루틴이 취소된 상태에서도 cleanup 작업을 안전하게 마칠 수 있습니다.

---

## 4. 결론 및 참고

- **NonCancellable의 활용:**
  - 취소 중에도 반드시 마쳐야 하는 정리(cleanup) 작업에 사용

- **참고 자료:**
  - [Kotlin 공식 문서: NonCancellable](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable/) 