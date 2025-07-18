# A02_CoroutineScope

## 1. CoroutineScope의 역할과 계층 구조

```kotlin
val scope = CoroutineScope(Job() + CoroutineName("My Scope"))
scope.launch(CoroutineName("Top-level Coroutine")) {
    delay(100)
    // ...
}
```
- **설명:**
  CoroutineScope는 코루틴의 생명주기와 계층 구조를 관리합니다. 부모 scope가 취소되면 모든 자식 코루틴도 함께 취소됩니다.

---

## 2. Scope 취소와 전파

```kotlin
val scope = CoroutineScope(Job())
val parent1 = scope.launch { /* ... */ }
val parent2 = scope.launch { /* ... */ }
scope.cancel() // 모든 자식, 손자 코루틴까지 취소
```
- **설명:**
  scope.cancel()을 호출하면 해당 scope에서 파생된 모든 코루틴이 재귀적으로 취소됩니다.

---

## 3. Scope 간 독립성

```kotlin
val scopeLeft = CoroutineScope(Job())
val scopeRight = CoroutineScope(Job())
// scopeLeft를 취소해도 scopeRight는 영향받지 않음
```
- **설명:**
  서로 다른 scope는 독립적으로 동작합니다. 하나의 scope를 취소해도 다른 scope에는 영향을 주지 않습니다.

---

## 4. 결론 및 참고

- **CoroutineScope의 활용:**
  - 구조적 동시성, 생명주기 관리, 안전한 취소 전파

- **참고 자료:**
  - [Kotlin 공식 문서: CoroutineScope](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#coroutine-scope) 