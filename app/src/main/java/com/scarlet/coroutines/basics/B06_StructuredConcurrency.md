# B06_StructuredConcurrency

## 1. 코루틴의 계층 구조와 구조적 동시성

```kotlin
runBlocking {
    launch {
        launch {
            launch { /* Level 3 */ }
            launch { /* Level 3 Another */ }
        }
    }
}
```
- **설명:**
  코루틴은 부모-자식 계층 구조를 가집니다. 부모가 취소되면 모든 자식도 함께 취소됩니다. 구조적 동시성은 예측 가능한 동작과 자원 관리를 가능하게 합니다.

---

## 2. 부모/자식 취소 전파 실험

```kotlin
val parent = launch {
    val child1 = launch { delay(1_000) }
    val child2 = launch { delay(1_000) }
    joinAll(child1, child2)
}
parent.cancelAndJoin() // 부모 취소 시 자식도 모두 취소
```
- **설명:**
  부모 코루틴을 취소하면 모든 자식 코루틴도 함께 취소됩니다. 반대로, 자식만 취소하면 부모와 다른 자식은 계속 동작합니다.

---

## 3. 예외 전파 실험

```kotlin
val parent = launch {
    val child1 = launch { throw RuntimeException("child 1 failed") }
    val child2 = launch { delay(1_000) }
    joinAll(child1, child2)
}
parent.join()
```
- **설명:**
  자식 코루틴에서 예외가 발생하면 부모와 형제 코루틴도 함께 취소될 수 있습니다. 구조적 동시성의 중요한 특징입니다.

---

## 4. 결론 및 참고

- **구조적 동시성의 장점:**
  - 예측 가능한 취소/예외 전파
  - 자원 누수 방지, 안전한 동시성

- **참고 자료:**
  - [Kotlin 공식 문서: Structured Concurrency](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#structured-concurrency) 