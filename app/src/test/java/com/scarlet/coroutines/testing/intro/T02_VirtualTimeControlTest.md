# T02_VirtualTimeControlTest

이 파일은 코루틴 테스트에서 가상 시간(Virtual Time) 제어 방법을 설명합니다. `StandardTestDispatcher`와 `UnconfinedTestDispatcher`의 차이점, `advanceTimeBy`, `runCurrent`, `advanceUntilIdle` 등의 시간 제어 함수들의 사용법을 보여줍니다.

---

## StandardTestDispatcher - 지연된 실행
```kotlin
@Test
fun `virtual time control - StandardTestDispatcher`() = runTest {
    var state = 0

    launch {
        state = 1
        delay(1_000)
        state = 2
    }

    // What `state` value should we compare with to make the test pass? 0 or 1?
    assertThat(state).isEqualTo(0)
    log("$currentTime")
}
```

### StandardTestDispatcher의 특징
- **지연된 실행**: 코루틴이 즉시 시작되지 않습니다.
- **명시적 진행**: `advanceTimeBy()`나 `advanceUntilIdle()` 호출이 필요합니다.
- **상태 확인**: launch 직후 state는 여전히 0입니다.
- **의도**: 시간 제어가 중요한 테스트에 적합합니다.

---

## UnconfinedTestDispatcher - 즉시 실행
```kotlin
@Test
fun `virtual time control - UnconfinedCoroutineDispatcher - eager`() =
    runTest(UnconfinedTestDispatcher()) {
        var state = 0

        launch {
            state = 1
            delay(1_000)
            state = 2
        }

        // What `state` value should we compare with to make the test pass? 0 or 1?
        // assertThat(state).isEqualTo(TODO())
        log("$currentTime")
    }
```

### UnconfinedTestDispatcher의 특징
- **즉시 실행**: 첫 번째 suspend point까지 즉시 실행됩니다.
- **Eager 실행**: state = 1이 즉시 실행되어 값이 변경됩니다.
- **상태 확인**: launch 직후 state는 1이 됩니다.
- **의도**: 실행 순서가 중요한 테스트에 적합합니다.

---

## 세밀한 시간 제어
```kotlin
@Test
fun `test virtual time control - StandardTestDispatcher`() = runTest {
    var count = 0

    launch {
        log("child start")
        delay(1_000)
        count = 1
        delay(1_000)
        count = 3
        delay(1_000)
        count = 5
        log("child end")
    }

    assertThat(count).isEqualTo(0)
    log("$currentTime")

//    advanceTimeBy(1_000);
//    log("$currentTime")
//    assertThat(count).isEqualTo(1)
//
//    advanceTimeBy(1_000); runCurrent()
//    log("$currentTime")
//    assertThat(count).isEqualTo(3)
//
//    advanceTimeBy(1_000); runCurrent()
//    log("$currentTime")
//    assertThat(count).isEqualTo(5)
}
```

### 시간 제어 함수들
1. **advanceTimeBy(time)**: 가상 시간을 진행시킵니다.
2. **runCurrent()**: 현재 시점에 예약된 작업들을 실행합니다.
3. **주석 해제 실험**: 주석을 해제하여 단계별 진행 확인이 가능합니다.

---

## runCurrent vs advanceUntilIdle
```kotlin
@Test
fun `runCurrent & advanceUntilIdle demo`() = runTest(UnconfinedTestDispatcher()) {
    var state = 0

    launch {
        state = 1
        delay(1_000)
        state = 2
        delay(1_000)
        state = 3
        delay(1_000)
        state = 4
    }

    assertThat(state).isEqualTo(1)
    log("$currentTime")

    // `runCurrent` run any tasks that are pending at or before the current virtual clock-time.
    // Calling this function will never advance the clock.
    advanceTimeBy(1_000); runCurrent()
    assertThat(state).isEqualTo(2)
    log("$currentTime")

    // Immediately execute all pending tasks and advance the virtual clock-time to the last delay.
    // If new tasks are scheduled due to advancing virtual time, they will be executed before
    // `advanceUntilIdle` returns.
    advanceUntilIdle()
    assertThat(state).isEqualTo(4)
    log("$currentTime")
}
```

### 함수별 특징

#### runCurrent()
- **시계 고정**: 가상 시간을 진행시키지 않습니다.
- **현재 작업 실행**: 현재 시점에 스케줄된 작업만 실행합니다.
- **세밀한 제어**: 단계별로 정확한 제어가 가능합니다.

#### advanceUntilIdle()
- **완전 진행**: 모든 대기 중인 작업이 완료될 때까지 진행합니다.
- **시간 자동 진행**: 마지막 delay까지 가상 시간을 자동으로 진행합니다.
- **편리한 완료**: 모든 작업의 완료를 한 번에 확인합니다.

---

## 실제적인 테스트 시나리오
```kotlin
@Test
fun `paused and resume dispatcher - realistic example`() = runTest {
    val list = mutableListOf<Int>().apply {
        add(42)
        launch {
            log(Thread.currentThread().name)
            add(777)
        }
    }

    assertThat(list).containsExactly(42)

    // How to make the test pass?
    // TODO()

    assertThat(list).containsExactly(42, 777)
}
```

### TODO 해결 방법들

#### 방법 1: runCurrent() 사용
```kotlin
runCurrent() // 대기 중인 코루틴 실행
```

#### 방법 2: advanceUntilIdle() 사용
```kotlin
advanceUntilIdle() // 모든 코루틴 완료까지 진행
```

#### 방법 3: 명시적 시간 진행
```kotlin
advanceTimeBy(1) // 최소한의 시간 진행
runCurrent()     // 현재 작업 실행
```

---

## 시간 제어 함수 비교표

| 함수 | 시간 진행 | 작업 실행 | 사용 목적 |
|------|----------|----------|----------|
| `advanceTimeBy(ms)` | O | X | 시간만 진행 |
| `runCurrent()` | X | O | 현재 작업만 실행 |
| `advanceUntilIdle()` | O | O | 모든 작업 완료 |

---

## 실험 시나리오

### 1. StandardTestDispatcher 실험
1. **기본 동작**: 주석된 코드 해제 전후 비교
2. **단계별 진행**: advanceTimeBy + runCurrent 조합
3. **상태 추적**: 각 단계에서 count 값 확인

### 2. UnconfinedTestDispatcher 실험
1. **즉시 실행**: state = 1이 즉시 반영되는지 확인
2. **지연 후 진행**: delay 이후의 상태 변화
3. **Thread 확인**: 로그에서 스레드 이름 확인

### 3. 혼합 시나리오 실험
1. **TODO 해결**: 다양한 방법으로 테스트 통과시키기
2. **성능 비교**: 각 방법의 실행 시간 측정
3. **복잡한 시나리오**: 중첩된 launch와 다양한 delay

---

## 실제 사용 패턴

### 빠른 완료 테스트
```kotlin
@Test
fun `fast completion test`() = runTest {
    val result = async { computeResult() }
    advanceUntilIdle()
    assertThat(result.await()).isEqualTo(expected)
}
```

### 단계별 검증 테스트
```kotlin
@Test
fun `step by step verification`() = runTest {
    startLongRunningProcess()
    
    advanceTimeBy(1000)
    runCurrent()
    assertThat(getProgress()).isEqualTo(25)
    
    advanceTimeBy(1000)
    runCurrent()
    assertThat(getProgress()).isEqualTo(50)
    
    advanceUntilIdle()
    assertThat(isCompleted()).isTrue()
}
```

---

## 결론 및 참고
- **TestDispatcher 선택:**
  - StandardTestDispatcher: 정밀한 시간 제어 필요시
  - UnconfinedTestDispatcher: 즉시 실행이 필요한 순서 테스트
- **시간 제어 전략:**
  - 간단한 테스트: advanceUntilIdle() 사용
  - 복잡한 테스트: advanceTimeBy() + runCurrent() 조합
  - 실시간 시뮬레이션: 단계별 시간 진행
- **참고 자료:**
  - [TestDispatchers guide](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/-test-dispatcher/)
  - [Virtual time control](https://kotlinlang.org/docs/coroutines-testing.html#virtual-time) 