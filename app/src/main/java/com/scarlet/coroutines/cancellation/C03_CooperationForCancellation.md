# C03_CooperationForCancellation

## 1. 협력적 취소(Cooperative Cancellation)와 isActive

```kotlin
private suspend fun printTwice() = withContext(Dispatchers.Default) {
    while (isActive) {
        log("job: I'm working..")
        delay(500)
    }
    log("job: I'm cancelled")
    cleanUp()
}
```
- **설명:**
  코루틴은 기본적으로 협력적으로 취소됩니다. isActive, ensureActive, yield, delay 등은 취소 신호를 감지해 코루틴을 안전하게 중단할 수 있게 해줍니다.

---

## 2. cleanup과 NonCancellable

```kotlin
private suspend fun cleanUp() {
    delay(100)
    log("Cleanup ...")
}
```
- **설명:**
  취소된 후에도 cleanup 작업을 하고 싶다면 NonCancellable 컨텍스트를 사용할 수 있습니다. 예제에서는 단순 delay로 cleanup을 수행합니다.

---

## 3. 결론 및 참고

- **협력적 취소의 활용:**
  - 무한 루프, 반복 작업 등에서 안전하게 취소를 감지하고 정리 작업 수행

- **참고 자료:**
  - [Kotlin 공식 문서: Cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html#cooperative-cancellation) 