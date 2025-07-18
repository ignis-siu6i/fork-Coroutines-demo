# A06_CoroutineScopeFunctions

## 1. coroutineScope와 supervisorScope

```kotlin
coroutineScope {
    // 자식 코루틴이 모두 끝날 때까지 현재 코루틴 일시 중단
}
supervisorScope {
    // 자식 중 하나가 실패해도 나머지는 계속 동작
}
```
- **설명:**
  coroutineScope는 자식이 모두 끝날 때까지 기다리며, 예외가 발생하면 전체가 취소됩니다. supervisorScope는 자식 중 하나가 실패해도 나머지는 계속 동작합니다.

---

## 2. withContext, withTimeout, withTimeoutOrNull

```kotlin
withContext(Dispatchers.IO) { /* ... */ }
withTimeout(1_000) { /* ... */ }
withTimeoutOrNull(1_000) { /* ... */ }
```
- **설명:**
  withContext는 코루틴 내에서 Dispatcher를 전환할 때, withTimeout/withTimeoutOrNull은 시간 제한 내에 작업을 완료하지 못하면 예외 또는 null을 반환합니다.

---

## 3. 실전 예제: 예외 전파와 안전한 동시 실행

```kotlin
private suspend fun getUserDetails(): Details = coroutineScope {
    val userName = async { getUserName() }
    val followersNumber = async { getFollowersNumber() }
    Details(userName.await(), followersNumber.await())
}
```
- **설명:**
  coroutineScope 내에서 async로 여러 작업을 동시에 실행하고, 하나라도 예외가 발생하면 전체가 취소됩니다. supervisorScope를 사용하면 일부만 실패해도 나머지는 계속 동작합니다.

---

## 4. 결론 및 참고

- **코루틴 스코프 함수의 활용:**
  - 구조적 동시성, 예외 전파, 안전한 동시 실행, 시간 제한 등

- **참고 자료:**
  - [Kotlin 공식 문서: coroutineScope, supervisorScope](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#coroutine-scope)
  - [Kotlin 공식 문서: withContext](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)
  - [Kotlin 공식 문서: withTimeout](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html) 