# A03_JobsRelation

## 1. Job의 계층 구조와 부모-자식 관계

```kotlin
val parentJob = launch(CoroutineName("Parent")) {
    launch { delay(1_000) } // Child1
    launch { delay(500) }   // Child2
}
```
- **설명:**
  Job은 부모-자식 계층 구조를 가집니다. 부모가 취소되면 모든 자식도 함께 취소됩니다. 자식이 먼저 끝나도 부모의 children에서 제외됩니다.

---

## 2. 부모가 자식의 완료를 기다림

```kotlin
val parent = launch {
    repeat(3) { i ->
        launch { delay((i + 1) * 200L) }
    }
    log("parent: I'm done, but will wait until all my children completes")
}
parent.join()
```
- **설명:**
  부모 Job은 자식 Job이 모두 끝날 때까지 자동으로 기다립니다. 명시적으로 join을 호출하지 않아도 구조적으로 보장됩니다.

---

## 3. Job의 독립성 부여

```kotlin
val scope = CoroutineScope(Job())
val parentJob = launch { /* ... */ }
val child = scope.launch(parentJob) { /* ... */ }
```
- **설명:**
  다른 scope나 Job을 명시적으로 지정하면, 부모-자식 관계를 커스텀할 수 있습니다. 이때 자식은 지정한 부모의 생명주기를 따릅니다.

---

## 4. 결론 및 참고

- **Job의 활용:**
  - 구조적 동시성, 취소 전파, 계층적 관리

- **참고 자료:**
  - [Kotlin 공식 문서: Job](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/) 