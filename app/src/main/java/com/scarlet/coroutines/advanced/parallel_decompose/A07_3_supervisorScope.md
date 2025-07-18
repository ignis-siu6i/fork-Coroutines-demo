# A07_3_supervisorScope

이 파일은 supervisorScope를 사용한 병렬 분해 패턴을 설명합니다. supervisorScope의 예외 격리 특성과 자식 코루틴 간의 독립적인 실패 처리, 다양한 예외 처리 시나리오를 예제로 보여줍니다.

---

## Using_supervisorScope.main
```kotlin
object Using_supervisorScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
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
  - supervisorScope 내에서 생성된 async 코루틴들은 root coroutine으로 취급됩니다.
  - 각 자식은 독립적으로 실패할 수 있으며, 한 자식의 실패가 다른 자식에게 전파되지 않습니다.
  - **의도:** supervisorScope의 기본 동작과 예외 격리를 실험적으로 보여줍니다.

---

## Parent_Cancellation_When_Using_supervisorScope.main
```kotlin
object Parent_Cancellation_When_Using_supervisorScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
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
  - 부모 코루틴이 취소되면 supervisorScope도 취소되고 내부의 모든 자식들도 취소됩니다.
  - supervisorScope는 자식 간의 예외 격리는 제공하지만 부모로부터의 취소는 여전히 전파됩니다.
  - **의도:** supervisorScope에서의 취소 전파 동작을 실험적으로 보여줍니다.

---

## Using_supervisorScope_and_when_child_failed1_not_calling_await.main
```kotlin
object Using_supervisorScope_and_when_child_failed1_not_calling_await {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
        // Root coroutines
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        // Exception is supposed to be thrown here when calling `await`.
        // But we are not calling `await` here to see what happens.

        Image("Fake Combined Image")
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
  - supervisorScope에서 자식이 실패해도 await()을 호출하지 않으면 예외가 무시됩니다.
  - root coroutine의 실패는 부모에게 전파되지 않습니다.
  - **의도:** supervisorScope에서 await() 없이 실패한 자식의 처리를 실험적으로 보여줍니다.

---

## Using_supervisorScope_and_when_child_failed2_calling_await.main
```kotlin
object Using_supervisorScope_and_when_child_failed2_calling_await {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
        // Root coroutines
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")

        // Exception will be thrown when calling `await`, and
        // will be rethrown by the supervisorScope unless caught here.
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
  - await()을 호출하면 실패한 자식의 예외가 supervisorScope로 다시 던져집니다.
  - supervisorScope는 이 예외를 다시 부모로 전파합니다.
  - **의도:** supervisorScope에서 await() 호출 시의 예외 전파를 실험적으로 보여줍니다.

---

## Using_supervisorScope_and_when_child_failed3_catch_outside_supervisorScope.main
```kotlin
object Using_supervisorScope_and_when_child_failed3_catch_outside_supervisorScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
        // Root coroutines
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
  - supervisorScope 밖에서 try-catch로 예외를 처리하는 올바른 패턴입니다.
  - 함수 호출 지점에서 예외를 처리하여 복구 로직을 구현할 수 있습니다.
  - **의도:** supervisorScope와 함께 사용하는 올바른 예외 처리 패턴을 실험적으로 보여줍니다.

---

## Using_supervisorScope_and_when_child_failed4_catch_inside_supervisorScope.main
```kotlin
object Using_supervisorScope_and_when_child_failed4_catch_inside_supervisorScope {
    private suspend fun loadAndCombine(name1: String, name2: String): Image = supervisorScope {
        // Root coroutines
        val apple = async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = async { loadImage(name2) }.onCompletion("kiwi")

        log("Waiting for two images to combine")
        try {
            combineImages(apple.await(), kiwi.await())
        } catch (e: Exception) {
            log("Caught $e in supervisorScope")
            if (e is CancellationException) throw e
            else Image("Fake Combined Image")
        }
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
  - supervisorScope 내부에서 try-catch로 예외를 처리하는 패턴입니다.
  - 함수 내부에서 예외를 처리하고 기본값을 반환할 수 있습니다.
  - **의도:** supervisorScope 내부에서의 예외 처리와 복구 전략을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **supervisorScope의 특징:**
  - 자식 코루틴 간의 예외 격리
  - Root coroutine으로 동작하는 자식들
  - 부모로부터의 취소는 여전히 전파
  - 명시적인 예외 처리 필요
- **사용 시나리오:**
  - 독립적인 병렬 작업들
  - 일부 실패를 허용하는 작업들
  - 개별 작업의 결과를 선택적으로 사용
- **참고 자료:**
  - [Kotlin 공식 문서: supervisorScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html) 