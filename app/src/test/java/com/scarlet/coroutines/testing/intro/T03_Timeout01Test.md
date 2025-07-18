# T03_Timeout01Test

이 파일은 코루틴에서 `withTimeout`을 사용한 타임아웃 처리와 이를 테스트하는 방법을 설명합니다. 가상 시간을 사용하여 빠르게 타임아웃 시나리오를 테스트하고, `async`와 함께 사용하는 패턴을 학습할 수 있습니다.

---

## 테스트 대상 함수
```kotlin
interface UserService {
    suspend fun load(): User
}

suspend fun loadUser(userService: UserService): User =
    withTimeout(5_000) {
        userService.load()
    }
```

### 함수 분석
- **withTimeout**: 5초 내에 완료되지 않으면 `TimeoutCancellationException` 발생
- **suspend 함수**: 코루틴 컨텍스트에서 실행
- **타임아웃 처리**: 네트워크 요청 등에서 무한 대기 방지

---

## 즉시 응답 테스트
```kotlin
@Test
fun `load responds immediately`() = runTest {
    coEvery { userService.load() } returns testUser

    val user = loadUser(userService)

    log("$currentTime")

    assertThat(user).isEqualTo(testUser)
}
```

### 테스트 특징
- **즉시 완료**: Mock이 지연 없이 즉시 응답
- **가상 시간**: `currentTime`이 0에 가까운 값
- **성공 케이스**: 타임아웃 없이 정상 완료
- **의도**: 빠른 응답에 대한 기본 동작 확인

---

## 타임아웃 직전 성공 테스트
```kotlin
@Test
fun `load in less than 5 seconds succeeds`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(4_999)
        testUser
    }

    val user = loadUser(userService)
    assertThat(user).isEqualTo(testUser)
    log("$currentTime")
}
```

### 테스트 특징
- **경계 조건**: 타임아웃 직전 (4,999ms)에 완료
- **가상 시간**: `runTest`로 인해 즉시 완료
- **성공 케이스**: 타임아웃 한계 내에서 성공
- **의도**: 타임아웃 경계값 테스트

---

## 타임아웃 발생 테스트
```kotlin
@Test(expected = TimeoutCancellationException::class)
fun `load timed out after 5 seconds`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(5_000)
        testUser
    }

    loadUser(userService)
}
```

### 테스트 특징
- **예외 발생**: `TimeoutCancellationException` 예상
- **정확한 타임아웃**: 5,000ms에서 타임아웃
- **실패 케이스**: 의도적인 타임아웃 테스트
- **의도**: 타임아웃 동작의 정확성 확인

---

## async와 함께 사용하는 타임아웃 테스트

### async 버전 함수
```kotlin
private fun CoroutineScope.loadUserAsync(userService: UserService): Deferred<User> = async {
    withTimeout(5_000) {
        userService.load()
    }
}
```

### async 성공 테스트
```kotlin
@Test
fun `testing async in time`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(4_999)
        testUser
    }

    val deferred = loadUserAsync(userService)

    val user = deferred.await()
    log("$currentTime")

    assertThat(user).isEqualTo(testUser)
}
```

### async 타임아웃 테스트
```kotlin
@Test(expected = TimeoutCancellationException::class)
fun `testing async timeout`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(5_000)
        testUser
    }

    val deferred = loadUserAsync(userService)

    deferred.await()
}
```

---

## 타임아웃 처리 패턴 비교

### 1. 직접 호출 패턴
```kotlin
suspend fun loadUser(): User = withTimeout(5_000) {
    userService.load()
}
```
**특징:**
- 간단한 구조
- 즉시 예외 발생
- 동기적 스타일

### 2. async 패턴
```kotlin
fun loadUserAsync(): Deferred<User> = async {
    withTimeout(5_000) {
        userService.load()
    }
}
```
**특징:**
- 비동기 실행
- await() 시점에 예외 처리
- 병렬 처리 가능

---

## 다양한 타임아웃 시나리오

### 1. 부분 성공 후 타임아웃
```kotlin
@Test(expected = TimeoutCancellationException::class)
fun `partial progress then timeout`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(2_000) // 일부 진행
        delay(4_000) // 총 6초로 타임아웃
        testUser
    }

    loadUser(userService)
}
```

### 2. 빠른 실패
```kotlin
@Test
fun `fast failure before timeout`() = runTest {
    coEvery { userService.load() } coAnswers {
        delay(1_000)
        throw IOException("Network error")
    }

    assertThrows<IOException> {
        runBlocking { loadUser(userService) }
    }
}
```

### 3. 취소와 타임아웃 조합
```kotlin
@Test
fun `cancellation vs timeout`() = runTest {
    val job = launch {
        loadUser(userService)
    }
    
    delay(3_000)
    job.cancel() // 타임아웃 전에 취소
    
    assertThrows<CancellationException> {
        job.join()
    }
}
```

---

## 실제 사용 시나리오

### 네트워크 요청 타임아웃
```kotlin
class UserRepository {
    suspend fun loadUser(id: String): Result<User> = try {
        val user = withTimeout(10_000) {
            apiService.getUser(id)
        }
        Result.success(user)
    } catch (e: TimeoutCancellationException) {
        Result.failure(NetworkTimeoutException("User load timeout"))
    }
}
```

### 병렬 요청의 타임아웃
```kotlin
suspend fun loadUserProfile(userId: String): UserProfile = coroutineScope {
    val userDeferred = async {
        withTimeout(5_000) { userService.getUser(userId) }
    }
    val postsDeferred = async {
        withTimeout(3_000) { postService.getUserPosts(userId) }
    }
    
    UserProfile(
        user = userDeferred.await(),
        posts = postsDeferred.await()
    )
}
```

---

## 실험 시나리오

### 1. 타임아웃 경계값 실험
1. **4,999ms**: 성공해야 함
2. **5,000ms**: 타임아웃 발생해야 함
3. **5,001ms**: 타임아웃 발생해야 함

### 2. 가상 시간 vs 실제 시간
1. **runTest 사용**: 즉시 완료되는 시간 확인
2. **runBlocking 사용**: 실제 시간 소요 확인
3. **성능 차이**: 테스트 실행 시간 비교

### 3. 예외 처리 실험
1. **try-catch**: TimeoutCancellationException 처리
2. **Result 타입**: 함수형 에러 처리
3. **상위 전파**: 예외가 상위로 전파되는 과정

---

## 모범 사례

### ✅ 권장: 적절한 타임아웃 설정
```kotlin
suspend fun loadCriticalData() = withTimeout(30_000) { // 30초
    heavyNetworkCall()
}

suspend fun loadNonCriticalData() = withTimeout(5_000) { // 5초
    lightNetworkCall()
}
```

### ✅ 권장: 타임아웃 예외 처리
```kotlin
suspend fun safeLoad(): Result<User> = try {
    Result.success(loadUser())
} catch (e: TimeoutCancellationException) {
    Result.failure(TimeoutException("Load timeout"))
}
```

### ✅ 권장: 가상 시간 테스트
```kotlin
@Test
fun `timeout test with virtual time`() = runTest {
    // 빠른 테스트 실행
    coEvery { service.load() } coAnswers { delay(6_000) }
    assertThrows<TimeoutCancellationException> {
        loadUser()
    }
}
```

### ❌ 피해야 할 패턴: 과도한 타임아웃
```kotlin
suspend fun badTimeout() = withTimeout(Long.MAX_VALUE) { // 너무 김
    networkCall()
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - withTimeout을 사용한 시간 제한 설정
  - TimeoutCancellationException 처리 방법
  - 가상 시간을 활용한 빠른 타임아웃 테스트
- **실제 적용:**
  - 네트워크 요청에 적절한 타임아웃 설정
  - 사용자 경험을 고려한 타임아웃 값 선택
  - 타임아웃 발생 시 적절한 사용자 피드백
- **참고 자료:**
  - [Kotlin 공식 문서: withTimeout](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html)
  - [Timeouts and cancellation](https://kotlinlang.org/docs/cancellation-and-timeouts.html) 