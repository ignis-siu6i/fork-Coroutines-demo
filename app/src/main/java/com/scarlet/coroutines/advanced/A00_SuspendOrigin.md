# A00_SuspendOrigin

## 1. 일반 함수와 suspend 함수의 차이

```kotlin
private fun fooWithDelay(a: Int, b: Int): Int {
    log("step 1")
    Thread.sleep(3_000)
    log("step 2")
    return a + b
}
```
- **설명:**
  일반 함수는 Thread.sleep 등으로 블로킹을 유발합니다. 이 방식은 메인 스레드나 UI 스레드에서 사용하면 앱이 멈추는 문제가 있습니다.

---

## 2. suspend 함수의 동작 원리

- **설명:**
  코루틴의 `suspend` 함수는 일시 중단(suspend)과 재개(resume)가 가능합니다. 내부적으로 상태 머신으로 변환되어, 중단점마다 스택을 보존하지 않고도 이어서 실행할 수 있습니다.

---

## 3. 결론 및 참고

- **suspend의 의의:**
  - 비동기/논블로킹 코드를 동기식처럼 작성 가능
  - 내부적으로 상태 머신으로 변환되어 효율적

- **참고 자료:**
  - [Kotlin 공식 문서: Suspend Functions](https://kotlinlang.org/docs/coroutines-basics.html#suspend-functions)
  - [Kotlin Coroutines: Under the hood](https://elizarov.medium.com/kotlin-coroutines-under-the-hood-4868d7abfc20) 