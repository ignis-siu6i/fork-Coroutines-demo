# A07_1_GlobalScope

이 파일은 GlobalScope 사용의 문제점과 왜 권장되지 않는지를 설명합니다. GlobalScope로 생성된 코루틴이 부모-자식 관계를 벗어나 독립적으로 동작하면서 발생하는 구조적 동시성 위반과 리소스 누수 문제를 다양한 예제로 보여줍니다.

---

## Works_But_Not_Recommended.main
```kotlin
object Works_But_Not_Recommended {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = GlobalScope.launch {
            image = loadAndCombine("apple", "kiwi")
            log("parent done.")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image")
    }
}
```
- **설명:**
  - GlobalScope.launch로 생성된 코루틴은 기술적으로는 동작하지만 구조적 동시성을 위반합니다.
  - 부모 코루틴과 독립적으로 실행되어 생명주기 관리가 어렵습니다.
  - **의도:** GlobalScope의 기본 동작은 되지만 권장되지 않는 이유를 실험적으로 보여줍니다.

---

## Even_If_Parent_Cancelled_Children_Keep_Going.main
```kotlin
object Even_If_Parent_Cancelled_Children_Keep_Going {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var image: Image? = null

        val parent = GlobalScope.launch {
            image = loadAndCombine("apple", "kiwi")
            log("parent done.")
        }.onCompletion("parent")

        delay(200)
        log("Cancel parent coroutine after 500ms")
        parent.cancelAndJoin()

        log("combined image = $image").also {
            delay(1_000) // To check what happens to children
        }
    }
}
```
- **설명:**
  - 부모 코루틴을 취소해도 GlobalScope로 생성된 자식 코루틴들은 계속 실행됩니다.
  - 이는 리소스 누수와 예상치 못한 동작을 일으킬 수 있습니다.
  - **의도:** GlobalScope의 구조적 동시성 위반과 취소 전파 실패를 실험적으로 보여줍니다.

---

## Even_If_One_Of_Children_Fails_Other_Child_Still_Runs.main
```kotlin
object Even_If_One_Of_Children_Fails_Other_Child_Still_Runs {
    private suspend fun loadAndCombineFail(name1: String, name2: String): Image {
        val apple = GlobalScope.async { loadImageFail(name1) }.onCompletion("apple")
        val kiwi = GlobalScope.async { loadImage(name2) }.onCompletion("kiwi")

        return combineImages(apple.await(), kiwi.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        onCompletion("runBlocking")
        var image: Image? = null

        val parent = GlobalScope.launch {
            image = loadAndCombineFail("apple", "kiwi")
            log("parent done.")
        }.onCompletion("parent")

        parent.join()
        log("combined image = $image").also {
            delay(1_000) // To check what happens to children
        }
    }
}
```
- **설명:**
  - GlobalScope로 생성된 코루틴 중 하나가 실패해도 다른 코루틴은 계속 실행됩니다.
  - 실패한 코루틴과 관련 없는 다른 코루틴이 불필요하게 계속 실행될 수 있습니다.
  - **의도:** GlobalScope에서 예외 격리로 인한 리소스 낭비 문제를 실험적으로 보여줍니다.

---

## 결론 및 참고
- **GlobalScope 사용을 피해야 하는 이유:**
  - 구조적 동시성 위반
  - 취소 전파 실패로 인한 리소스 누수
  - 생명주기 관리의 어려움
  - 예상치 못한 백그라운드 실행
- **대안:**
  - coroutineScope, supervisorScope 사용
  - 적절한 CoroutineScope 전달
- **참고 자료:**
  - [Kotlin 공식 문서: GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/) 