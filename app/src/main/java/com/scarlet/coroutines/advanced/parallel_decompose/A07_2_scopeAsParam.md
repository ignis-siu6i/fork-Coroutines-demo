# A07_2_scopeAsParam

이 파일은 CoroutineScope를 매개변수로 전달하는 패턴의 장단점을 설명합니다. 이 방법이 GlobalScope보다는 낫지만 여전히 권장되지 않는 이유와, 부모-자식 관계에서의 취소 전파와 예외 처리 문제를 다양한 예제로 보여줍니다.

---

## Passing_Coroutine_Scope_As_Parameter_Works_But_Not_Recommended.main
```kotlin
object Passing_Coroutine_Scope_As_Parameter_Works_But_Not_Recommended {
    private suspend fun loadAndCombine(scope: CoroutineScope, name1: String, name2: String): Image {
        val apple = scope.async { loadImage(name1) }.onCompletion("apple")
        val kiwi = scope.async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        return combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine(this, "apple", "kiwi")
            log("parent done: image = $image")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - CoroutineScope를 매개변수로 전달하여 부모의 스코프에서 자식 코루틴을 생성합니다.
  - GlobalScope보다는 나은 접근법이지만 여전히 권장되지 않습니다.
  - **의도:** 스코프 전달 패턴의 기본 동작과 한계를 실험적으로 보여줍니다.

---

## Parent_Cancellation_When_Passing_Coroutine_Scope_As_Parameter.main
```kotlin
object Parent_Cancellation_When_Passing_Coroutine_Scope_As_Parameter {
    private suspend fun loadAndCombine(scope: CoroutineScope, name1: String, name2: String): Image {
        val apple = scope.async { loadImage(name1) }.onCompletion("apple")
        val kiwi = scope.async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        return combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine(this, "apple", "kiwi")
            log("Parent done: image = $image")
        }.onCompletion("parent")

        delay(200)
        parent.cancel(CancellationException("Cancel parent coroutine after 500ms"))
        parent.join()

        log("combined image = $image").also {
            delay(1_000) // To check what happens to children just in case
        }
    }
}
```
- **설명:**
  - 부모 코루틴이 취소되면 전달된 스코프로 생성된 자식 코루틴들도 함께 취소됩니다.
  - 취소 전파는 올바르게 동작하지만 다른 문제들이 여전히 존재합니다.
  - **의도:** 스코프 전달 패턴에서의 취소 전파 동작을 실험적으로 보여줍니다.

---

## Child_Failure_When_Passing_Coroutine_Scope_As_Parameter.main
```kotlin
object Child_Failure_When_Passing_Coroutine_Scope_As_Parameter {
    private suspend fun loadAndCombine(scope: CoroutineScope, name1: String, name2: String): Image {
        // Are these Root Coroutines because they are created through `scope`?
        // If not root coroutines, try-catch around `await()` is eventually useless.
        val apple = scope.async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = scope.async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        // Documentation says try-catch is useless here in case of non-root coroutines.
        // However, it is executed anyway.
        // You'd better think of it as covered, but not treated as handled ... [by Jungsun Kim]
        val image = try {
            combineImages(
                apple.await(),
                kiwi.await(),
            )
        } catch (e: Exception) { // eventually useless
            log("combineImages: Caught $e")
            if (e is CancellationException) throw e
            Image("Fake Combined Image")
        }
        return image
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = launch {
            try {
                image = loadAndCombine(this, "apple", "kiwi")
                log("Parent done: image = $image") // is this reachable or not?
            } catch (e: Exception) { // eventually useless
                log("Caught in parent: $e")
                if (e is CancellationException) throw e
                image = Image("Oops") // useless
            }
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image") // <== unreadable!
    }
}
```
- **설명:**
  - 전달된 스코프로 생성된 자식 코루틴에서 예외가 발생하면 예외 처리가 복잡해집니다.
  - 함수 내부의 try-catch가 예상대로 동작하지 않을 수 있습니다.
  - 예외가 부모로 전파되어 전체 구조가 취소될 수 있습니다.
  - **의도:** 스코프 전달 패턴에서의 예외 처리 복잡성과 예측 어려움을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **스코프 매개변수 전달의 문제점:**
  - 예외 처리의 복잡성과 예측 어려움
  - 함수의 책임 범위 모호화
  - 구조적 동시성 원칙 위반
  - 테스트와 디버깅의 어려움
- **더 나은 대안:**
  - coroutineScope { } 사용
  - supervisorScope { } 사용
  - 적절한 스코프 함수 선택
- **참고 자료:**
  - [Structured Concurrency](https://elizarov.medium.com/structured-concurrency-722d765aa952) 