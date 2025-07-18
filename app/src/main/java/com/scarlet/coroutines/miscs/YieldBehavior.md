# YieldBehavior

이 파일은 yield() 함수의 동작과 CoroutineStart 모드의 차이점을 설명합니다. DEFAULT vs UNDISPATCHED 시작 모드에서 yield()가 어떻게 다르게 동작하는지, 그리고 다양한 Dispatcher와의 상호작용을 보여줍니다.

---

## Default_Starting_Mode.main
```kotlin
object Default_Starting_Mode {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var state = 0

        // Change the dispatcher to Dispatchers.Unconfined
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

        log("parent before delay: state = $state")
        delay(500)
        log("parent after delay: state = $state")
    }
}
```
- **설명:**
  - `CoroutineStart.DEFAULT` 모드에서는 코루틴이 즉시 스케줄링되지만 현재 스레드를 블로킹하지 않습니다.
  - `yield()`는 다른 코루틴에게 실행 기회를 주고 현재 코루틴을 일시 중단합니다.
  - `Dispatchers.Default`에서는 yield()가 정상적으로 동작합니다.
  - **의도:** 기본 시작 모드에서의 yield() 동작과 실행 순서를 실험적으로 보여줍니다.

---

## Undispatched_Starting_Mode.main
```kotlin
object Undispatched_Starting_Mode {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        var state = 0

        // Change the dispatcher to Dispatchers.Unconfined
        launch(context = Dispatchers.Default, start = CoroutineStart.UNDISPATCHED) {
            state = 1
            log("child before yield: state = $state")
            yield()
            state = 2
            log("child after yield: state = $state")
            delay(1_000)
            state = 3
            log("child after 1000ms: state = $state")
        }

        log("parent before delay: state = $state")
        delay(500)
        log("parent after delay: state = $state")
    }
}
```
- **설명:**
  - `CoroutineStart.UNDISPATCHED` 모드에서는 첫 번째 suspend point까지 현재 스레드에서 즉시 실행됩니다.
  - `yield()`가 첫 번째 suspend point가 되어 그 이후부터 Dispatcher가 적용됩니다.
  - state = 1까지는 부모와 같은 스레드에서 실행되어 즉시 반영됩니다.
  - **의도:** UNDISPATCHED 모드에서의 즉시 실행과 yield() 이후의 동작 변화를 실험적으로 보여줍니다.

---

## CoroutineStart 모드 비교

### DEFAULT 모드
```kotlin
launch(start = CoroutineStart.DEFAULT) {
    // 즉시 스케줄링되지만 dispatcher에 의해 실행됨
    println("This may not run immediately")
}
```

### UNDISPATCHED 모드
```kotlin
launch(start = CoroutineStart.UNDISPATCHED) {
    // 첫 suspend point까지 현재 스레드에서 즉시 실행
    println("This runs immediately")
    yield() // 여기서부터 dispatcher 적용
    println("This follows dispatcher rules")
}
```

### LAZY 모드
```kotlin
val job = launch(start = CoroutineStart.LAZY) {
    // start() 또는 join() 호출 전까지 실행되지 않음
    println("This runs only when explicitly started")
}
job.start() // 또는 job.join()
```

---

## Dispatcher별 yield() 동작

### Dispatchers.Default
```kotlin
launch(Dispatchers.Default) {
    yield() // 다른 코루틴에게 기회를 줌
}
```

### Dispatchers.Unconfined
```kotlin
launch(Dispatchers.Unconfined) {
    yield() // 효과가 제한적일 수 있음
}
```

### Dispatchers.Main (Android)
```kotlin
launch(Dispatchers.Main) {
    yield() // UI 스레드에서 다른 작업에게 기회를 줌
}
```

---

## 실험 시나리오
1. **기본 동작 확인**: 
   - DEFAULT 모드에서 state 변화 시점 관찰
   - yield() 전후의 로그 순서 확인

2. **UNDISPATCHED 모드 비교**:
   - state = 1이 즉시 반영되는지 확인
   - yield() 이후 동작 차이 관찰

3. **Dispatcher 변경 실험**:
   - 주석에 있는 대로 `Dispatchers.Unconfined`로 변경
   - yield() 동작 차이 확인

---

## yield() 함수의 용도
1. **협조적 멀티태스킹**: 긴 작업 중간에 다른 코루틴에게 기회 제공
2. **취소 확인**: yield()는 취소 상태도 확인함
3. **성능 최적화**: CPU 집약적 작업에서 응답성 향상

---

## 실용적 사용 예제
```kotlin
suspend fun longRunningTask() {
    repeat(1000) { i ->
        // CPU 집약적 작업
        heavyComputation(i)
        
        // 주기적으로 yield하여 다른 코루틴에게 기회 제공
        if (i % 100 == 0) {
            yield()
        }
    }
}
```

---

## 결론 및 참고
- **CoroutineStart 모드의 특징:**
  - DEFAULT: 일반적인 비동기 시작
  - UNDISPATCHED: 즉시 실행으로 성능 최적화
  - LAZY: 지연 시작으로 메모리 효율성
- **yield()의 활용:**
  - 협조적 멀티태스킹
  - 취소 확인 포인트
  - 응답성 향상
- **참고 자료:**
  - [Kotlin 공식 문서: CoroutineStart](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-start/)
  - [yield() function](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html) 