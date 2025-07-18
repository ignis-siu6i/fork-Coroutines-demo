# A07_4_coroutineScope

이 파일은 coroutineScope를 사용한 병렬 분해 패턴을 설명합니다. coroutineScope의 구조적 동시성, 예외 전파 특성, 올바른 예외 처리 방법과 잘못된 예외 처리 패턴을 다양한 예제로 보여줍니다.

---

## Using_coroutineScope.main
```kotlin
object Using_coroutineScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        // Root coroutines
        val apple = async { loadImage(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine("apple", "kiwi")
            log("Parent done: image = $image")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - coroutineScope는 구조적 동시성을 보장하는 권장되는 병렬 분해 패턴입니다.
  - 내부에서 생성된 async 코루틴들은 모두 완료되거나 하나라도 실패하면 전체가 취소됩니다.
  - **의도:** coroutineScope의 기본 동작과 구조적 동시성을 실험적으로 보여줍니다.

---

## Parent_Cancellation_When_Using_coroutineScope.main
```kotlin
object Parent_Cancellation_When_Using_coroutineScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        val apple = async { loadImage(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine("apple", "kiwi")
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
  - 부모 코루틴이 취소되면 coroutineScope와 모든 자식 코루틴들이 함께 취소됩니다.
  - 구조적 동시성에 의해 취소가 올바르게 전파됩니다.
  - **의도:** coroutineScope에서의 취소 전파와 구조적 동시성을 실험적으로 보여줍니다.

---

## Using_coroutineScope_and_when_child_failed1_not_calling_await.main
```kotlin
object Using_coroutineScope_and_when_child_failed1_not_calling_await {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        // Since non-root coroutines, exceptions will be thrown inside `async` block
        // even if we are not calling `await`.
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        Image("Fake Combined Image")
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine("apple", "kiwi")
            log("Parent done: image = $image")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - coroutineScope에서는 자식 코루틴이 실패하면 await()을 호출하지 않아도 즉시 예외가 전파됩니다.
  - non-root coroutine의 특성으로 예외가 즉시 부모로 전파됩니다.
  - **의도:** coroutineScope에서 await() 없이도 발생하는 즉시 예외 전파를 실험적으로 보여줍니다.

---

## Using_coroutineScope_and_when_child_failed2_calling_await.main
```kotlin
object Using_coroutineScope_and_when_child_failed2_calling_await {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine("apple", "kiwi")
            log("Parent done: image = $image")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - await()을 호출할 때도 실패한 자식의 예외가 coroutineScope로 전파됩니다.
  - 예외는 더 이상 지연되지 않고 즉시 처리되어야 합니다.
  - **의도:** coroutineScope에서 await() 호출 시의 예외 전파를 실험적으로 보여줍니다.

---

## Using_coroutineScope_and_when_child_failed3_catch_outside_coroutineScope.main
```kotlin
object Using_coroutineScope_and_when_child_failed3_catch_outside_coroutineScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = launch {
            try {
                image = loadAndCombine("apple", "kiwi")
                log("Parent done: image = $image")
            } catch (e: Exception) {
                log("Caught $e in parent")
                if (e is CancellationException) throw e
                image = Image("Oops")
            }
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - coroutineScope 밖에서 try-catch로 예외를 처리하는 올바른 패턴입니다.
  - 함수 호출 지점에서 예외를 처리하여 안전하게 복구할 수 있습니다.
  - **의도:** coroutineScope와 함께 사용하는 올바른 예외 처리 패턴을 실험적으로 보여줍니다.

---

## Using_coroutineScope_and_when_child_failed4_inside_coroutineScope_wrong_way.main
```kotlin
object Using_coroutineScope_and_when_child_failed4_inside_coroutineScope_wrong_way {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = coroutineScope {
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        try {
            combineImages(apple.await(), kiwi.await())
        } catch (e: Exception) { // useless
            log("Caught in coroutineScope: $e")
            if (e is CancellationException) throw e
            Image("Oops")
        }
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = launch {
            image = loadAndCombine("apple", "kiwi")
            log("Parent done: image = $image") // <== unreachable
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - coroutineScope 내부에서 try-catch를 사용하는 잘못된 패턴입니다.
  - non-root coroutine에서는 예외가 즉시 전파되어 try-catch가 무의미할 수 있습니다.
  - 이 코드는 예상대로 동작하지 않을 수 있습니다.
  - **의도:** coroutineScope 내부에서의 잘못된 예외 처리 패턴과 그 문제점을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **coroutineScope의 특징:**
  - 구조적 동시성 보장
  - 즉시 예외 전파 (non-root coroutines)
  - 모든 자식 완료 대기
  - 하나라도 실패하면 전체 취소
- **올바른 사용 패턴:**
  - 스코프 밖에서 예외 처리
  - 함수 호출 지점에서 try-catch 사용
  - 구조적 동시성 활용
- **권장 시나리오:**
  - 모든 작업이 성공해야 하는 경우
  - 엄격한 실패 정책이 필요한 경우
  - 구조적 동시성을 보장하고 싶은 경우
- **참고 자료:**
  - [Kotlin 공식 문서: coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 