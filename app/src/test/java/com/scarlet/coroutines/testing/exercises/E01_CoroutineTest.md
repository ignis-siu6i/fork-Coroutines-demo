# E01_CoroutineTest

이 파일은 코루틴 테스트의 기본적인 연습 문제를 제공합니다. `StandardTestDispatcher`와 `UnconfinedTestDispatcher`의 차이점, `coroutineScope`를 사용한 구조화된 동시성을 실제 테스트 시나리오로 학습할 수 있습니다.

---

## 테스트 대상 클래스들
```kotlin
interface UserRepo {
    suspend fun register(user: String)
    suspend fun getAllUsers(): List<String>
}

class FakeUserRepo : UserRepo {
    private val users = mutableListOf<String>()

    override suspend fun register(user: String) {
        users.add(user)
    }

    override suspend fun getAllUsers(): List<String> {
        return users
    }
}
```

### 클래스 설명
- **UserRepo**: 사용자 등록과 조회를 위한 인터페이스
- **FakeUserRepo**: 테스트용 메모리 기반 구현체
- **suspend 함수**: 실제 네트워크나 데이터베이스 작업을 시뮬레이션

---

## 문제 1: UnconfinedTestDispatcher 사용
```kotlin
// How to make this test pass?
@Test
fun `should register user`() = runTest(UnconfinedTestDispatcher()) {
    // Given
    // When
    launch {
        userRepo.register("Lindsay Wagner")
    }
    launch {
        userRepo.register("Diane Lane")
    }

    // Then
    assertThat(userRepo.getAllUsers()).containsExactly("Lindsay Wagner", "Diane Lane")
}
```

### 해결 방법 분석
- **문제**: StandardTestDispatcher(기본값)를 사용하면 launch 블록이 즉시 실행되지 않음
- **해결**: `runTest(UnconfinedTestDispatcher())`로 즉시 실행 보장
- **핵심**: UnconfinedTestDispatcher는 첫 번째 suspend point까지 즉시 실행
- **결과**: register 호출이 즉시 완료되어 assertion 통과

---

## 문제 2: launch에서 UnconfinedTestDispatcher 개별 지정
```kotlin
@Test
fun `should register user2`() = runTest {
    // Given
    // When
    launch(UnconfinedTestDispatcher()) {
        userRepo.register("Lindsay Wagner")
    }
    launch(UnconfinedTestDispatcher()) {
        userRepo.register("Diane Lane")
    }

    // Then
    assertThat(userRepo.getAllUsers()).containsExactly("Lindsay Wagner", "Diane Lane")
}
```

### 해결 방법 분석
- **접근**: 각 launch 블록에 개별적으로 UnconfinedTestDispatcher 지정
- **효과**: 해당 launch만 즉시 실행, 다른 코루틴은 기본 동작 유지
- **장점**: 세밀한 제어 가능
- **단점**: 반복적인 코드 작성 필요

---

## 문제 3: coroutineScope 사용 (권장 방법)
```kotlin
@Test
fun `should register user3`() = runTest {
    // Given
    // When
    coroutineScope {
        launch {
            userRepo.register("Lindsay Wagner")
        }
        launch {
            userRepo.register("Diane Lane")
        }
    }

    // Then
    assertThat(userRepo.getAllUsers()).containsExactly("Lindsay Wagner", "Diane Lane")
}
```

### 해결 방법 분석
- **구조화된 동시성**: coroutineScope가 모든 자식 코루틴 완료를 보장
- **자동 대기**: coroutineScope 블록이 끝나면 모든 launch 완료 보장
- **깔끔한 코드**: 추가 디스패처 지정 불필요
- **권장 패턴**: 가장 읽기 쉽고 안전한 방법

---

## 각 접근법의 장단점 비교

### 1. UnconfinedTestDispatcher를 runTest에 전달
```kotlin
runTest(UnconfinedTestDispatcher()) { /* ... */ }
```
**장점:**
- 전체 테스트가 즉시 실행
- 간단한 설정

**단점:**
- 모든 코루틴이 즉시 실행되어 제어가 어려움
- 복잡한 테스트에서는 예상과 다른 동작

### 2. 개별 launch에 UnconfinedTestDispatcher 지정
```kotlin
launch(UnconfinedTestDispatcher()) { /* ... */ }
```
**장점:**
- 세밀한 제어 가능
- 필요한 부분만 즉시 실행

**단점:**
- 반복적인 코드
- 코루틴마다 개별 설정 필요

### 3. coroutineScope 사용 (권장)
```kotlin
coroutineScope { launch { /* ... */ } }
```
**장점:**
- 구조화된 동시성 보장
- 모든 자식 코루틴 완료까지 자동 대기
- 깔끔하고 읽기 쉬운 코드
- 예외 처리도 자동으로 구조화됨

**단점:**
- coroutineScope의 개념 이해 필요

---

## 실험 시나리오

### 1. StandardTestDispatcher vs UnconfinedTestDispatcher
1. **기본 테스트**: runTest()만 사용했을 때 실패 확인
2. **즉시 실행**: UnconfinedTestDispatcher 사용으로 성공 확인
3. **로그 추가**: 실행 순서 관찰을 위한 로그 추가

### 2. 다양한 해결 방법 비교
1. **방법별 실행**: 세 가지 방법 모두 실행해보기
2. **성능 측정**: 각 방법의 실행 시간 비교
3. **복잡도 증가**: 더 많은 launch 블록 추가 시 동작 확인

### 3. 실제 비동기 작업 추가
```kotlin
// suspend 함수에 delay 추가
override suspend fun register(user: String) {
    delay(100) // 네트워크 지연 시뮬레이션
    users.add(user)
}
```

---

## 모범 사례

### ✅ 권장: coroutineScope 사용
```kotlin
@Test
fun `structured concurrency test`() = runTest {
    coroutineScope {
        launch { longRunningTask1() }
        launch { longRunningTask2() }
        launch { longRunningTask3() }
    }
    // 모든 작업 완료 보장
    verifyResults()
}
```

### ⚠️ 주의: 디스패처 남용
```kotlin
@Test
fun `avoid dispatcher overuse`() = runTest {
    // 모든 곳에 UnconfinedTestDispatcher 사용하지 말 것
    launch(UnconfinedTestDispatcher()) { /* ... */ }
    launch(UnconfinedTestDispatcher()) { /* ... */ }
    launch(UnconfinedTestDispatcher()) { /* ... */ }
}
```

### ❌ 피해야 할 패턴
```kotlin
@Test
fun `problematic pattern`() = runTest {
    launch { register("user1") }
    launch { register("user2") }
    // coroutineScope 없이는 완료 보장 안됨
    verify() // 실패 가능성 높음
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - TestDispatcher의 종류와 동작 차이 이해
  - 구조화된 동시성의 중요성
  - 테스트에서 코루틴 완료를 보장하는 방법
- **권장 패턴:**
  - 대부분의 경우 `coroutineScope` 사용
  - 복잡한 시나리오에서는 `advanceUntilIdle()` 활용
  - 명확한 의도가 있을 때만 특정 TestDispatcher 지정
- **참고 자료:**
  - [Structured Concurrency](https://kotlinlang.org/docs/composing-suspending-functions.html#structured-concurrency-with-async)
  - [Testing coroutines](https://kotlinlang.org/docs/coroutines-testing.html) 