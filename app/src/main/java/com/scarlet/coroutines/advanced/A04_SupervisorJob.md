# A04_SupervisorJob

## 1. SupervisorJob의 기본 동작

```kotlin
val scope = CoroutineScope(SupervisorJob())
val child1 = scope.launch { /* ... */ }
val child2 = scope.launch { /* ... */ }
scope.cancel() // 모든 자식이 함께 취소됨
```
- **설명:**
  SupervisorJob은 일반 Job과 달리 자식 중 하나가 실패해도 다른 자식에게 영향을 주지 않습니다. 하지만 scope 자체를 취소하면 모든 자식이 함께 취소됩니다.

---

## 2. 자식 개별 취소와 에러 격리

```kotlin
val scope = CoroutineScope(SupervisorJob())
val child1 = scope.launch { throw RuntimeException("child 1 failed") }
val child2 = scope.launch { /* ... */ }
// child1만 실패해도 child2는 계속 동작
```
- **설명:**
  SupervisorJob을 사용하면 자식 코루틴의 실패가 부모나 형제에게 전파되지 않아, 에러 격리가 가능합니다.

---

## 3. 결론 및 참고

- **SupervisorJob의 활용:**
  - 독립적인 자식 코루틴 관리, 에러 격리

- **참고 자료:**
  - [Kotlin 공식 문서: SupervisorJob](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job/)
  - [공식 가이드: Structured Concurrency](https://kotlinlang.org/docs/exception-handling.html#supervision) 