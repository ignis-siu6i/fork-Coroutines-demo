# YieldBehavior

## 1. yield와 CoroutineStart 모드

```kotlin
launch(context = Dispatchers.Default, start = CoroutineStart.DEFAULT) {
    state = 1
    log("child before yield: state = $state")
    yield()
    state = 2
    log("child after yield: state = $state")
    delay(1_000)
    state = 3
    log("child after 1000ms: state = $state")
}
```
- **설명:**
  yield는 현재 코루틴의 실행을 일시 중단하고, 다른 코루틴에 실행 기회를 양보합니다. CoroutineStart 모드(Default, Undispatched 등)에 따라 yield의 동작이 달라질 수 있습니다.

---

## 2. Dispatcher에 따른 yield의 효과

- **설명:**
  Dispatchers.Default와 Dispatchers.Unconfined 등 Dispatcher 종류에 따라 yield가 실제로 스케줄링에 미치는 영향이 다릅니다. Unconfined는 yield를 무시할 수 있습니다.

---

## 3. 결론 및 참고

- **yield의 활용:**
  - 협력적 멀티태스킹, 코루틴 간 실행 양보, 테스트 등

- **참고 자료:**
  - [Kotlin 공식 문서: yield](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/yield.html) 