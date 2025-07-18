# B00_WhyCoroutine

## 1. 동기식 호출의 한계

```kotlin
private fun postItem(item: Item) {
    val token = requestToken()
    val post = createPost(token, item)
    showPost(post)
}
```
- **설명:**
  동기적으로 네트워크 요청을 처리하면 전체 프로그램이 멈추고, UI가 멈추거나 여러 작업을 동시에 처리할 수 없습니다.

---

## 2. 콜백 지옥(Callback Hell)

```kotlin
private fun loadData() {
    networkRequest { data ->
        anotherRequest(data) { data1 ->
            anotherRequest(data1) { data2 ->
                // ... (중략)
                anotherRequest(data9) {
                    println(it)
                }
            }
        }
    }
}
```
- **설명:**
  비동기 처리를 위해 콜백을 중첩해서 사용하면 코드가 복잡해지고, 가독성이 떨어집니다. 에러 처리, 흐름 제어가 매우 어려워집니다.

---

## 3. CompletableFuture, Rx, 그리고 코루틴

### CompletableFuture 예시

```kotlin
private fun postItem(item: Item): CompletableFuture<Void> {
    return requestToken()
        .thenComposeAsync { token -> createPost(token, item) }
        .thenAcceptAsync { post -> showPost(post) }
}
```
- **설명:**
  Java의 CompletableFuture를 사용하면 콜백 지옥을 어느 정도 피할 수 있지만, 여전히 체이닝이 복잡하고 예외 처리도 어렵습니다.

### Rx 예시

```kotlin
private fun requestToken(): Observable<Token> = Observable.create { emitter ->
    // ...
}
```
- **설명:**
  RxJava를 사용하면 스트림 기반으로 비동기 처리가 가능하지만, 연산자 학습 곡선이 높고, 복잡한 흐름에서는 디버깅이 어렵습니다.

---

## 4. 코루틴의 간결함

```kotlin
private suspend fun postItem(item: Item) {
    val token = requestToken()
    val post = createPost(token, item)
    showPost(post)
}
```
- **설명:**
  코루틴을 사용하면 동기식 코드와 거의 동일한 형태로 비동기 처리를 할 수 있습니다. `suspend` 함수로 선언하면, 내부에서 `delay`나 네트워크 요청 등 비동기 작업을 자연스럽게 처리할 수 있습니다.

---

## 5. 결론

- **코루틴의 장점:**
  - 동기식 코드처럼 읽기 쉽고, 유지보수성이 높음
  - 콜백 지옥, 복잡한 체이닝 없이 비동기 흐름 제어 가능
  - 예외 처리, 취소, 구조적 동시성 등 고급 기능 지원

- **참고 자료:**
  - [Kotlin 공식 문서: Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
  - [코루틴과 RxJava 비교](https://developer.android.com/kotlin/coroutines/coroutines-adv#rxjava) 