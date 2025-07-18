# C01_Cancellation

이 파일은 코루틴 취소의 기본 원리와 계층 구조에서의 취소 전파를 설명합니다. 부모-자식 관계에서 취소가 어떻게 전파되는지, 다양한 취소 방법(cancel, cancelAndJoin, cancelChildren)의 차이점을 다양한 시나리오로 보여줍니다.

---

## Cancel_Parent_Scope.main
```kotlin
object Cancel_Parent_Scope {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())

        var child: Job? = null
        val parent = scope.launch {
            child = launch {
                delay(1_000)
                log("child is done")
            }.onCompletion("child")
        }.onCompletion("parent")

        delay(100)

        scope.cancel()

        log("parent cancelled = ${parent.isCancelled}")
        log("child cancelled = ${child?.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - 최상위 스코프를 취소하면 모든 하위 코루틴들이 취소됩니다.
  - scope.cancel()은 scope의 Job과 모든 자식 Job들을 취소합니다.
  - 계층 구조: scope(❌) → parent(❌) → child(❌)
  - **의도:** 스코프 레벨에서의 전체 취소와 계층적 취소 전파를 실험적으로 보여줍니다.

---

## Cancel_Parent_Coroutine.main
```kotlin
object Cancel_Parent_Coroutine {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())

        var child1: Job? = null
        var child2: Job? = null
        val parentJob = scope.launch {
            child1 = launch {
                log("child1 started")
                delay(1_000)
            }.onCompletion("child1")
            child2 = launch {
                log("child2 started")
                delay(1_000)
            }.onCompletion("child2")
        }.onCompletion("parentJob")

        delay(200)

        parentJob.cancelAndJoin()

        log("parent job cancelled = ${parentJob.isCancelled}")
        log("child1 job cancelled = ${child1?.isCancelled}")
        log("child2 job cancelled = ${child2?.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - 부모 코루틴을 취소하면 모든 자식 코루틴도 취소됩니다.
  - cancelAndJoin()은 취소 후 완료될 때까지 기다립니다.
  - 계층 구조: scope(✅) → parent(❌) → child1(❌), child2(❌)
  - **의도:** 부모 코루틴 취소가 자식들에게 미치는 영향을 실험적으로 보여줍니다.

---

## Cancel_Child_Coroutine.main
```kotlin
object Cancel_Child_Coroutine {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())

        var child1: Job? = null
        var child2: Job? = null
        val parentJob = scope.launch {
            child1 = launch {
                log("child1 started")
                delay(1_000)
            }.onCompletion("child1")
            child2 = launch {
                log("child2 started")
                delay(1_000)
            }.onCompletion("child2")
        }.onCompletion("parentJob")

        delay(200)

        child1?.cancel()
        parentJob.join()

        log("parent job cancelled = ${parentJob.isCancelled}")
        log("child1 job cancelled = ${child1?.isCancelled}")
        log("child2 job cancelled = ${child2?.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - 자식 코루틴 하나를 취소해도 부모나 다른 자식에게는 영향을 주지 않습니다.
  - 취소는 아래로만 전파되고 위로는 전파되지 않습니다.
  - 계층 구조: scope(✅) → parent(✅) → child1(❌), child2(✅)
  - **의도:** 자식 코루틴 취소의 격리성과 단방향 전파를 실험적으로 보여줍니다.

---

## Cancel_Children_Only_To_Reuse_Parent_Job.main
```kotlin
object Cancel_Children_Only_To_Reuse_Parent_Job {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())

        var child1: Job? = null
        var child2: Job? = null
        val parentJob = scope.launch {
            child1 = launch {
                log("child1 started")
                delay(1_000)
            }.onCompletion("child1")
            child2 = launch {
                log("child2 started")
                delay(1_000)
            }.onCompletion("child2")
        }.onCompletion("parentJob")

        delay(200)

        parentJob.cancelChildren()
        parentJob.join()

        log("parent job cancelled = ${parentJob.isCancelled}")
        log("child1 job cancelled = ${child1?.isCancelled}")
        log("child2 job cancelled = ${child2?.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - cancelChildren()은 자식들만 취소하고 부모 Job은 유지합니다.
  - 부모 Job을 재사용하여 새로운 자식들을 시작할 수 있습니다.
  - 계층 구조: scope(✅) → parent(✅) → child1(❌), child2(❌)
  - **의도:** 선택적 취소를 통한 Job 재사용 패턴을 실험적으로 보여줍니다.

---

## Cancel_Children_Only_To_Reuse_Scope.main
```kotlin
object Cancel_Children_Only_To_Reuse_Scope {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())

        var child1: Job? = null
        var child2: Job? = null
        val parentJob = scope.launch {
            child1 = launch {
                log("child1 started")
                delay(1_000)
            }.onCompletion("child1")
            child2 = launch {
                log("child2 started")
                delay(1_000)
            }.onCompletion("child2")
        }.onCompletion("parentJob")

        delay(200)

        scope.coroutineContext.job.cancelChildren()
        parentJob.join()

        log("parent job cancelled = ${parentJob.isCancelled}")
        log("child1 job cancelled = ${child1?.isCancelled}")
        log("child2 job cancelled = ${child2?.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - 스코프 레벨에서 cancelChildren()을 호출하여 모든 직계 자식만 취소합니다.
  - 스코프 자체는 유지되어 새로운 코루틴을 시작할 수 있습니다.
  - 계층 구조: scope(✅) → parent(❌) → child1(❌), child2(❌)
  - **의도:** 스코프 레벨에서의 선택적 취소와 스코프 재사용을 실험적으로 보여줍니다.

---

## Cancel_Parent_Job_Quiz.main
```kotlin
object Cancel_Parent_Job_Quiz {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        val scope = CoroutineScope(Job())
        val job = Job()

        // Who's child's parent?
        val child = scope.launch(job) {
            delay(1_000)
        }.onCompletion("child")

        delay(100)

        // How to cancel the child via its parent?
        // job.cancelAndJoin() or scope.coroutineContext.job.cancelAndJoin()?
        scope.coroutineContext.job.cancelAndJoin()
        // job.cancelAndJoin()

        delay(1_000)
        log("child cancelled = ${child.isCancelled}")
        scope.completeStatus("scope")
    }
}
```
- **설명:**
  - scope.launch(job)에서 실제 부모는 누구인지 퀴즈 형태로 제시합니다.
  - job 매개변수로 전달되었지만 실제 부모-자식 관계는 scope입니다.
  - 올바른 취소 방법은 scope.coroutineContext.job.cancelAndJoin()입니다.
  - **의도:** 코루틴 부모-자식 관계의 미묘한 차이와 정확한 취소 방법을 퀴즈를 통해 실험적으로 보여줍니다.

---

## 취소 전파 규칙 요약
1. **하향 전파**: 부모 취소 → 모든 자식 취소
2. **격리**: 자식 취소 → 부모/형제에 영향 없음
3. **선택적 취소**: cancelChildren()으로 자식만 취소
4. **스코프 관리**: 적절한 레벨에서 취소하여 재사용성 확보

---

## 결론 및 참고
- **코루틴 취소의 핵심 원리:**
  - 계층적 구조와 단방향 전파
  - 구조적 동시성을 통한 안전한 리소스 관리
  - 다양한 취소 방법의 적절한 선택
  - Job 재사용을 통한 효율적인 코루틴 관리
- **참고 자료:**
  - [Kotlin 공식 문서: Cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html)
  - [Structured Concurrency](https://elizarov.medium.com/structured-concurrency-722d765aa952) 