# CvtToSuspendingFunctionTest

이 파일은 콜백 기반 API를 suspend 함수로 변환하는 방법과 이를 테스트하는 시나리오를 설명합니다. Retrofit의 `Call` 인터페이스를 `suspendCoroutine`을 사용하여 suspend 함수로 변환하고, 취소 처리까지 포함한 완전한 마이그레이션 방법을 학습할 수 있습니다.

---

## 테스트 설정
```kotlin
class CvtToSuspendingFunctionTest {

    @MockK
    lateinit var mockApi: RecipeApi

    @MockK(relaxed = true) // or use @RelaxedMockK
    lateinit var mockCall: Call<List<Recipe>>

    @MockK
    lateinit var mockResponse: Response<List<Recipe>>

    @Before
    fun init() {
        MockKAnnotations.init(this)
        every { mockApi.search(any(), any()) } returns mockCall
    }
}
```

### Mock 설정 분석
- **RecipeApi**: Retrofit API 인터페이스 모킹
- **Call<List<Recipe>>**: Retrofit Call 객체 모킹
- **Response<List<Recipe>>**: HTTP 응답 객체 모킹
- **relaxed = true**: 모든 함수에 대해 기본값 반환

---

## 콜백 기반 API 테스트
```kotlin
@Test
fun `Callback - should return valid recipes`() {
    // Arrange (Given)
    every { mockResponse.isSuccessful } returns true
    every { mockResponse.body() } returns mRecipes
    every { mockCall.enqueue(any()) } answers {
        val callback = firstArg<Callback<List<Recipe>>>()
        callback.onResponse(mockCall, mockResponse)
    }

    val target = UsingCallback_Demo2

    // Act (When)
    target.searchRecipes("eggs", mockApi, object : RecipeCallback<List<Recipe>> {
        override fun onSuccess(response: Resource<List<Recipe>>) {
            // Assert (Then)
            assertThat(response).isEqualTo(Resource.Success(mRecipes))
        }

        override fun onError(response: Resource<List<Recipe>>) {
            fail("Should not be called")
        }
    })
}
```

### 콜백 테스트 특징
- **Mock 응답 설정**: 성공적인 HTTP 응답 시뮬레이션
- **enqueue 모킹**: `answers` 블록으로 콜백 즉시 호출
- **성공 검증**: onSuccess 콜백에서 결과 검증
- **실패 방지**: onError가 호출되면 테스트 실패

---

## Suspend 함수 변환 테스트
```kotlin
@Test
fun `Suspending Function - should return valid recipes`() = runTest {
    // Arrange (Given)
    every { mockResponse.isSuccessful } returns true
    every { mockResponse.body() } returns mRecipes
    every { mockCall.enqueue(any()) } answers {
        val callback = firstArg<Callback<List<Recipe>>>()
        callback.onResponse(mockCall, mockResponse)
    }

    val target = CvtToSuspendingFunction_Demo2

    // Act (When)
    val response = target.searchRecipes("eggs", mockApi, TODO())

    // Assert (Then)
    assertThat(response).isEqualTo(Resource.Success(mRecipes))
}
```

### Suspend 함수 테스트 특징
- **runTest 사용**: 코루틴 테스트 환경
- **직접 호출**: 콜백 없이 직접 결과 반환
- **가상 시간**: 즉시 완료되는 비동기 작업
- **TODO 해결**: 실제 구현에서는 취소 토큰 전달

---

## 취소 처리 테스트
```kotlin
@Test
fun `Suspending Function - should cancel searchRecipes`() = runBlocking {
    // Arrange (Given)
    every { mockResponse.isSuccessful } returns true
    every { mockResponse.body() } returns mRecipes
    every {
        mockCall.enqueue(any())
    } answers {
        val callback = firstArg<Callback<List<Recipe>>>()
        newSingleThreadScheduledExecutor().schedule({
            callback.onResponse(mockCall, mockResponse)
        }, 1, TimeUnit.SECONDS)
    }

    val target = CvtToSuspendingFunction_Demo2

    // Act (When)
    val job = launch {
        target.searchRecipes("eggs", mockApi, TODO())
    }

    delay(500)
    job.cancelAndJoin()

    // Assert (Then)
    verify { mockCall.cancel() }
}
```

### 취소 테스트 특징
- **지연된 응답**: 1초 후 응답하도록 설정
- **조기 취소**: 500ms 후 코루틴 취소
- **취소 검증**: `mockCall.cancel()` 호출 확인
- **runBlocking 사용**: 실제 시간 기반 테스트

---

## suspendCoroutine을 사용한 변환 구현

### 기본 변환 패턴
```kotlin
suspend fun <T> Call<T>.await(): T = suspendCoroutine { continuation ->
    enqueue(object : Callback<T> {
        override fun onResponse(call: Call<T>, response: Response<T>) {
            if (response.isSuccessful) {
                continuation.resume(response.body()!!)
            } else {
                continuation.resumeWithException(
                    HttpException(response)
                )
            }
        }

        override fun onFailure(call: Call<T>, t: Throwable) {
            continuation.resumeWithException(t)
        }
    })
}
```

### 취소 지원 변환 패턴
```kotlin
suspend fun <T> Call<T>.awaitWithCancellation(): T = suspendCancellableCoroutine { continuation ->
    // 취소 시 Call 취소
    continuation.invokeOnCancellation {
        this.cancel()
    }
    
    enqueue(object : Callback<T> {
        override fun onResponse(call: Call<T>, response: Response<T>) {
            if (response.isSuccessful) {
                continuation.resume(response.body()!!)
            } else {
                continuation.resumeWithException(
                    HttpException(response)
                )
            }
        }

        override fun onFailure(call: Call<T>, t: Throwable) {
            continuation.resumeWithException(t)
        }
    })
}
```

---

## 실제 구현 예제

### RecipeRepository 변환 전
```kotlin
class RecipeRepository(private val api: RecipeApi) {
    fun searchRecipes(
        query: String,
        callback: RecipeCallback<List<Recipe>>
    ) {
        api.search(query, "your-api-key").enqueue(object : Callback<List<Recipe>> {
            override fun onResponse(call: Call<List<Recipe>>, response: Response<List<Recipe>>) {
                if (response.isSuccessful) {
                    callback.onSuccess(Resource.Success(response.body()!!))
                } else {
                    callback.onError(Resource.Error("API Error"))
                }
            }

            override fun onFailure(call: Call<List<Recipe>>, t: Throwable) {
                callback.onError(Resource.Error(t.message ?: "Unknown error"))
            }
        })
    }
}
```

### RecipeRepository 변환 후
```kotlin
class RecipeRepository(private val api: RecipeApi) {
    suspend fun searchRecipes(query: String): Resource<List<Recipe>> = try {
        val response = api.search(query, "your-api-key").awaitWithCancellation()
        Resource.Success(response)
    } catch (e: Exception) {
        Resource.Error(e.message ?: "Unknown error")
    }
}
```

---

## Mock 설정 패턴

### 성공 응답 모킹
```kotlin
every { mockResponse.isSuccessful } returns true
every { mockResponse.body() } returns testData
every { mockCall.enqueue(any()) } answers {
    val callback = firstArg<Callback<List<Recipe>>>()
    callback.onResponse(mockCall, mockResponse)
}
```

### 실패 응답 모킹
```kotlin
every { mockResponse.isSuccessful } returns false
every { mockResponse.code() } returns 404
every { mockCall.enqueue(any()) } answers {
    val callback = firstArg<Callback<List<Recipe>>>()
    callback.onResponse(mockCall, mockResponse)
}
```

### 네트워크 오류 모킹
```kotlin
every { mockCall.enqueue(any()) } answers {
    val callback = firstArg<Callback<List<Recipe>>>()
    callback.onFailure(mockCall, IOException("Network error"))
}
```

### 지연 응답 모킹
```kotlin
every { mockCall.enqueue(any()) } answers {
    val callback = firstArg<Callback<List<Recipe>>>()
    // 별도 스레드에서 지연 후 응답
    GlobalScope.launch {
        delay(1000)
        callback.onResponse(mockCall, mockResponse)
    }
}
```

---

## 실험 시나리오

### 1. TODO 해결하기
1. **TODO 위치 파악**: 테스트 코드의 TODO() 부분 찾기
2. **취소 토큰 구현**: CancellationToken이나 적절한 취소 메커니즘 추가
3. **테스트 통과**: 모든 테스트가 통과하도록 구현

### 2. 다양한 응답 시나리오 테스트
1. **성공 케이스**: 정상적인 데이터 응답
2. **HTTP 오류**: 404, 500 등 HTTP 에러 코드
3. **네트워크 오류**: 연결 실패, 타임아웃 등
4. **빈 응답**: null이나 빈 리스트 응답

### 3. 취소 동작 검증
1. **즉시 취소**: 요청 직후 즉시 취소
2. **지연 후 취소**: 일정 시간 후 취소
3. **완료 후 취소**: 요청 완료 후 취소 시도

---

## 마이그레이션 모범 사례

### ✅ 권장: suspendCancellableCoroutine 사용
```kotlin
suspend fun <T> Call<T>.await(): T = suspendCancellableCoroutine { continuation ->
    continuation.invokeOnCancellation { cancel() }
    // 콜백 구현
}
```

### ✅ 권장: 에러 처리 개선
```kotlin
suspend fun safeApiCall(): Resource<Data> = try {
    Resource.Success(api.getData().await())
} catch (e: HttpException) {
    Resource.Error("HTTP ${e.code()}: ${e.message()}")
} catch (e: IOException) {
    Resource.Error("Network error: ${e.message}")
}
```

### ✅ 권장: 점진적 마이그레이션
```kotlin
// 1단계: suspend 함수 추가 (기존 콜백 유지)
suspend fun searchRecipesSuspend(query: String) = // new implementation
fun searchRecipes(query: String, callback: Callback) = // existing implementation

// 2단계: 콜백 함수를 suspend 함수로 위임
fun searchRecipes(query: String, callback: Callback) {
    GlobalScope.launch {
        try {
            val result = searchRecipesSuspend(query)
            callback.onSuccess(result)
        } catch (e: Exception) {
            callback.onError(e)
        }
    }
}
```

### ❌ 피해야 할 패턴: 취소 처리 누락
```kotlin
suspend fun badConversion(): Data = suspendCoroutine { continuation ->
    // invokeOnCancellation 누락 - 취소 처리 안됨
    api.getData().enqueue(callback)
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - suspendCoroutine vs suspendCancellableCoroutine 차이
  - 콜백 기반 API의 suspend 함수 변환 방법
  - 취소 처리를 포함한 완전한 마이그레이션
- **실제 적용:**
  - 기존 Retrofit 콜백 API의 suspend 함수 변환
  - RxJava에서 코루틴으로의 마이그레이션
  - 레거시 비동기 코드의 점진적 개선
- **참고 자료:**
  - [suspendCoroutine 공식 문서](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/suspend-coroutine.html)
  - [Migrating to Kotlin coroutines](https://developer.android.com/kotlin/coroutines/coroutines-adv) 