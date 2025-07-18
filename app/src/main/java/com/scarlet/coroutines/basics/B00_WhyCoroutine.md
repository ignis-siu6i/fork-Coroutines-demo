# B00_WhyCoroutine

이 파일은 동기/비동기/콜백/CompletableFuture/Rx/코루틴 등 다양한 비동기 처리 패턴을 실제 코드로 비교합니다. 각 main 함수는 실무에서 마주치는 비동기 처리의 복잡성과, 코루틴이 제공하는 간결함·가독성·유지보수성의 장점을 실험적으로 보여줍니다.

---

## UsingSyncCall.main
```kotlin
object UsingSyncCall {
    @JvmStatic
    fun main(args: Array<String>) {
        postItem(Item("kiwi"))
        loop()
    }
}
```
- **설명:**
  - 동기식(Blocking) 네트워크 요청을 순차적으로 처리합니다. `requestToken()`과 `createPost()`가 모두 sleep으로 블로킹됩니다.
  - loop()는 별도의 스레드가 아닌 메인 스레드에서 실행되어, 네트워크 요청이 끝나야만 실행됩니다.
  - **의도:** 동기식 코드의 한계(전체 프로그램이 멈춤, UI 멈춤, 동시성 불가)를 보여줍니다.

---

## UsingCallback.main
```kotlin
object UsingCallback {
    @JvmStatic
    fun main(args: Array<String>) {
        postItem(Item("Kiwi"))
        loop()
    }
}
```
- **설명:**
  - 비동기 처리를 위해 콜백을 중첩해서 사용합니다. 각 네트워크 요청이 별도 스레드에서 실행되어, 메인 스레드가 블로킹되지 않습니다.
  - 하지만 콜백 중첩이 깊어질수록 코드가 복잡해지고, 에러 처리·흐름 제어가 어려워집니다.
  - **의도:** 콜백 패턴의 복잡성과 한계를 보여줍니다.

---

## CallbackHell.main
```kotlin
object CallbackHell {
    @JvmStatic
    fun main(args: Array<String>) {
        loadData()
    }
}
```
- **설명:**
  - 콜백 지옥(Callback Hell)의 극단적 예시입니다. 10단계 이상의 중첩 콜백이 발생하며, 코드 가독성이 매우 떨어집니다.
  - **의도:** 콜백 패턴이 깊어질수록 유지보수와 에러 처리가 얼마나 어려워지는지 실감할 수 있습니다.

---

## AsyncWithCompletableFuture.main
```kotlin
object AsyncWithCompletableFuture {
    @JvmStatic
    fun main(args: Array<String>) {
        postItem(Item("Kiwi"))
        loop()
    }
}
```
- **설명:**
  - Java의 CompletableFuture를 사용해 비동기 처리를 체이닝합니다. thenComposeAsync, thenAcceptAsync 등으로 콜백 지옥을 어느 정도 피할 수 있습니다.
  - 하지만 여전히 예외 처리, 흐름 제어가 복잡하며, 코드가 장황해집니다.
  - **의도:** CompletableFuture의 장점과 한계를 모두 보여줍니다.

---

## AsyncWithRx.main
```kotlin
object AsyncWithRx {
    @JvmStatic
    fun main(args: Array<String>) {
        postItem(Item("Kiwi"))
        loop()
    }
}
```
- **설명:**
  - RxJava를 사용해 Observable 체인으로 비동기 처리를 구현합니다. flatMap, subscribeOn, observeOn 등으로 흐름을 제어합니다.
  - 연산자 학습 곡선이 높고, 복잡한 흐름에서는 디버깅이 어렵습니다.
  - **의도:** Rx 패턴의 장점(스트림 기반, 체이닝)과 한계(복잡성, 디버깅)를 보여줍니다.

---

## AsyncWithCoroutine.main
```kotlin
object AsyncWithCoroutine {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        launch {
            postItem(Item("Kiwi"))
        }
        launch {
            loop()
        }
    }
}
```
- **설명:**
  - 코루틴(suspend 함수)을 사용해 동기식 코드와 거의 동일한 형태로 비동기 처리를 구현합니다.
  - launch로 여러 코루틴을 동시에 실행할 수 있고, delay로 논블로킹 대기를 자연스럽게 처리합니다.
  - **의도:** 코루틴이 제공하는 코드의 간결함, 가독성, 유지보수성, 동시성의 장점을 실감할 수 있습니다.

---

## 결론 및 참고
- **코루틴의 장점:**
  - 동기식 코드처럼 읽기 쉽고, 유지보수성이 높음
  - 콜백 지옥, 복잡한 체이닝 없이 비동기 흐름 제어 가능
  - 예외 처리, 취소, 구조적 동시성 등 고급 기능 지원
- **참고 자료:**
  - [Kotlin 공식 문서: Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
  - [코루틴과 RxJava 비교](https://developer.android.com/kotlin/coroutines/coroutines-adv#rxjava) 