# B06_StructuredConcurrency

이 파일은 구조적 동시성(Structured Concurrency) 개념을 설명합니다. 부모-자식 코루틴 관계, 취소 전파, 예외 발생 시의 동작 등 코루틴 계층 구조와 그 의미를 다양한 예제로 보여줍니다.

---

## Nested_Coroutines.main
```kotlin
object Nested_Coroutines {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        log("Top-Level Coroutine")
        launch {
            log("\tLevel 1 Coroutine")
            launch {
                log("\t\tLevel 2 Coroutine")
                launch { log("\t\t\tLevel 3 Coroutine") }
                launch { log("\t\t\tLevel 3 Another Coroutine") }
            }
        }
    }
}
```
- **설명:**
  - 코루틴의 계층 구조(부모-자식-손자)를 시각적으로 보여줍니다.
  - 각 코루틴이 계층적으로 생성되고, 부모가 끝나야 자식도 끝납니다.
  - **의도:** 구조적 동시성의 기본 구조와 계층적 실행을 실험적으로 보여줍니다.

---

## Canceling_parent_coroutine_cancels_the_parent_and_its_children.main
```kotlin
object Canceling_parent_coroutine_cancels_the_parent_and_its_children {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val parent = launch {
            val child1 = launch { delay(1_000) }
            val child2 = launch { delay(1_000) }
            joinAll(child1, child2)
        }
        parent.cancelAndJoin() // 부모 취소 시 자식도 모두 취소
        log("Done")
    }
}
```
- **설명:**
  - 부모 코루틴을 취소하면 모든 자식 코루틴도 함께 취소됩니다.
  - **의도:** 구조적 동시성에서 취소 전파의 기본 원리를 실험적으로 보여줍니다.

---

## Canceling_a_child_cancels_only_the_child.main
```kotlin
object Canceling_a_child_cancels_only_the_child {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var child1: Job? = null
        val parent = launch {
            child1 = launch { delay(1_000) }
            val child2 = launch { delay(1_000) }
            joinAll(child1, child2)
        }
        delay(500)
        child1?.cancel()
        parent.join()
        log("Done")
    }
}
```
- **설명:**
  - 자식 코루틴(child1)만 취소하면, 부모와 다른 자식(child2)은 계속 동작합니다.
  - **의도:** 구조적 동시성에서 자식 취소의 범위와 전파를 실험적으로 보여줍니다.

---

## Failed_parent_causes_cancellation_of_all_children.main
```kotlin
object Failed_parent_causes_cancellation_of_all_children {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val parent = launch {
            launch { delay(1_000) }
            launch { delay(1_000) }
            throw RuntimeException("parent failed")
        }
        parent.join()
        log("Done.")
    }
}
```
- **설명:**
  - 부모 코루틴에서 예외가 발생하면, 모든 자식 코루틴도 함께 취소됩니다.
  - **의도:** 예외 전파와 구조적 동시성의 안전성을 실험적으로 보여줍니다.

---

## Failed_child_causes_cancellation_of_its_parent_and_siblings.main
```kotlin
object Failed_child_causes_cancellation_of_its_parent_and_siblings {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val parent = launch {
            val child1 = launch { throw RuntimeException("child 1 failed") }
            val child2 = launch { delay(1_000) }
            joinAll(child1, child2)
        }
        parent.join()
        log("Done.")
    }
}
```
- **설명:**
  - 자식 코루틴(child1)에서 예외가 발생하면, 부모와 형제(child2)도 함께 취소됩니다.
  - **의도:** 구조적 동시성에서 예외 전파의 범위와 안전성을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **구조적 동시성의 장점:**
  - 예측 가능한 취소/예외 전파
  - 자원 누수 방지, 안전한 동시성
- **참고 자료:**
  - [Kotlin 공식 문서: Structured Concurrency](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#structured-concurrency) 