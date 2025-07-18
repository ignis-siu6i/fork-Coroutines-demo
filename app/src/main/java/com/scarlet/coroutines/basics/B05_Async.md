# B05_Async

이 파일은 async/await를 이용한 비동기 결과 수집, GlobalScope.async의 위험성 등을 예제로 설명합니다. 각 main 함수는 async의 올바른 사용법과 실무에서 주의할 점을 실험적으로 보여줍니다.

---

## Async_Demo1.main
```kotlin
object Async_Demo1 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferred = async {
            log("\tRequest user with Id A001")
            getUser("A001")
        }
        log("Waiting for results ...")
        val user = deferred.await()
        log("Returned user = $user")
        log("Done")
    }
}
```
- **설명:**
  - async로 비동기 작업을 시작하고, await로 결과를 기다립니다. launch와 달리 async는 값을 반환(Deferred)합니다.
  - log 순서를 통해 async/await의 비동기-동기 전환을 명확히 보여줍니다.
  - **의도:** async/await 패턴의 기본 동작과, launch와의 차이(값 반환)를 실험적으로 보여줍니다.

---

## Async_Demo2.main
```kotlin
@DelicateCoroutinesApi
object Async_Demo2 {
    @ExperimentalStdlibApi
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val deferred = GlobalScope.async {
            log("\tRequest user with Id A001")
            getUser("A001")
        }
        log("Waiting for results ...")
        val user = deferred.await()
        log("Returned user = $user")
        log("Done")
    }
}
```
- **설명:**
  - GlobalScope.async는 앱 전체 생명주기와 무관하게 동작합니다. runBlocking이 끝나도 async의 결과를 안전하게 수집하지 못할 수 있습니다.
  - **의도:** GlobalScope.async의 위험성(메모리 누수, 예측 불가한 동작, 결과 수집의 어려움)을 보여줍니다.

---

## 결론 및 참고
- **async/await의 용도:**
  - 비동기적으로 값을 반환받고 싶을 때 사용
  - launch와 달리 결과(Deferred)를 다룸
- **참고 자료:**
  - [Kotlin 공식 문서: async](https://kotlinlang.org/docs/coroutines-basics.html#async-coroutine-builder) 