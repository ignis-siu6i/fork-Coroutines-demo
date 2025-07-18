# C01_Cancellation

## 1. 코루틴의 취소와 전파

```kotlin
val scope = CoroutineScope(Job())
val parent = scope.launch {
    val child = launch {
        delay(1_000)
        log("child is done")
    }
}
scope.cancel() // 부모 scope를 취소하면 모든 자식도 함께 취소
```
- **설명:**
  코루틴의 Job에는 cancel 메서드가 있어, 부모가 취소되면 모든 자식 코루틴도 함께 취소됩니다. 취소된 Job은 더 이상 자식 코루틴의 부모가 될 수 없습니다.

---

## 2. 부모/자식/형제 간 취소 실험

```kotlin
val parentJob = scope.launch {
    val child1 = launch { delay(1_000) }
    val child2 = launch { delay(1_000) }
}
parentJob.cancelAndJoin() // 부모만 취소하면 자식도 모두 취소

child1?.cancel() // 자식만 취소하면 부모와 다른 자식은 계속 동작
```
- **설명:**
  부모를 취소하면 모든 자식이 함께 취소되고, 자식만 취소하면 부모와 다른 자식은 영향을 받지 않습니다.

---

## 3. cancelChildren, cancelAndJoin, cancel 후 상태

```kotlin
parentJob.cancelChildren() // 자식만 취소, 부모는 재사용 가능
scope.coroutineContext.job.cancelChildren() // scope의 모든 자식만 취소
```
- **설명:**
  cancelChildren을 사용하면 부모/스코프는 유지하면서 자식만 취소할 수 있습니다. 취소 후 상태를 로그로 확인할 수 있습니다.

---

## 4. 결론 및 참고

- **코루틴 취소의 활용:**
  - 구조적 동시성, 자원 누수 방지, 안전한 취소 전파

- **참고 자료:**
  - [Kotlin 공식 문서: Cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html) 