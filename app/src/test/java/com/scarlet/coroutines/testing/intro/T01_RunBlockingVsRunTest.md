# T01_RunBlockingVsRunTest

이 파일은 코루틴 테스트에서 `runBlocking`과 `runTest`의 차이점을 설명합니다. 실제 시간 vs 가상 시간의 차이, 테스트 성능과 안정성에 미치는 영향을 Mock을 사용한 실제 예제로 보여줍니다.

---

## 테스트 설정 (공통)
```kotlin
@ExperimentalCoroutinesApi
class RunBlockingVsRunTest {

    interface ArticleService {
        suspend fun getArticle(id: String): Article
    }

    class Repository(private val articleService: ArticleService) {
        suspend fun getArticle(id: String): Article {
            return articleService.getArticle(id)
        }
    }

    // SUT (System Under Test)
    private lateinit var repository: Repository
    private val expectedArticle = Article("A006", "Roman Elizarov", "Kotlin Coroutines")

    @MockK
    private lateinit var mockArticleService: ArticleService

    @Before
    fun init() {
        MockKAnnotations.init(this)
        repository = Repository(mockArticleService)
    }
}
```

### 설정 설명
- **MockK**: Kotlin을 위한 모킹 라이브러리 사용
- **Repository 패턴**: ArticleService를 의존성으로 받는 Repository
- **coEvery**: suspend 함수를 모킹하기 위한 MockK 함수

---

## runBlocking 테스트
```kotlin
@Test
fun `runBlocking demo`() = runBlocking {
    // Given
    coEvery {
        mockArticleService.getArticle(any())
    } coAnswers {
        delay(2_000) // fake network delay
        expectedArticle
    }

    val duration = measureTimeMillis {
        // When
        val article = repository.getArticle("A006")
        // Then
        assertThat(article).isEqualTo(expectedArticle)
    }

    log("time elapsed = $duration")
}
```

### runBlocking의 특징
- **실제 시간**: delay(2_000)이 실제로 2초 동안 실행됩니다.
- **블로킹**: 테스트가 완료될 때까지 스레드를 블로킹합니다.
- **성능**: 느린 테스트 실행 시간 (약 2초)
- **용도**: 실제 시간이 중요한 통합 테스트

---

## runTest 테스트
```kotlin
@Test
fun `runTest demo`() = runTest {
    // Given
    coEvery {
        mockArticleService.getArticle(any())
    } coAnswers {
        delay(2_000) // fake network delay
        expectedArticle
    }

    val duration = measureTimeMillis {
        // When
        val article = repository.getArticle("A001")
        // Then
        assertThat(article).isEqualTo(expectedArticle)
    }

    log("time elapsed = $duration")
}
```

### runTest의 특징
- **가상 시간**: delay(2_000)이 즉시 완료됩니다.
- **논블로킹**: 가상 시간으로 빠른 실행
- **성능**: 매우 빠른 테스트 실행 시간 (몇 밀리초)
- **용도**: 유닛 테스트와 빠른 피드백이 필요한 테스트

---

## 실행 결과 비교

### runBlocking 결과
```
time elapsed = 2003  // 실제로 2초 소요
```

### runTest 결과
```
time elapsed = 5     // 몇 밀리초만 소요
```

---

## 언제 어떤 것을 사용할까?

### runBlocking 사용 시나리오
1. **통합 테스트**: 실제 네트워크나 데이터베이스 호출
2. **실제 시간 의존성**: 타임아웃이나 실제 지연이 중요한 테스트
3. **레거시 코드**: 기존 블로킹 코드와의 호환성
4. **성능 측정**: 실제 실행 시간 측정이 필요한 경우

```kotlin
@Test
fun `integration test with real timing`() = runBlocking {
    // 실제 API 호출이나 데이터베이스 접근
    val result = realApiService.getData()
    // 실제 시간이 중요한 검증
}
```

### runTest 사용 시나리오 (권장)
1. **유닛 테스트**: Mock을 사용한 빠른 테스트
2. **시간 제어**: 가상 시간으로 빠른 실행
3. **대부분의 코루틴 테스트**: 일반적인 suspend 함수 테스트
4. **CI/CD**: 빠른 테스트 실행이 중요한 환경

```kotlin
@Test
fun `fast unit test with virtual time`() = runTest {
    // Mock을 사용한 빠른 테스트
    val result = repositoryWithMock.getData()
    // 즉시 완료되는 검증
}
```

---

## Mock 설정 패턴

### coEvery와 coAnswers 사용
```kotlin
coEvery {
    mockService.getData(any())
} coAnswers {
    delay(1000)  // 가상 지연
    testData
}
```

### 다양한 Mock 응답
```kotlin
// 성공 케이스
coEvery { mockService.getData("success") } returns successData

// 실패 케이스
coEvery { mockService.getData("error") } throws RuntimeException("Network error")

// 지연 응답
coEvery { mockService.getData("slow") } coAnswers {
    delay(5000)
    slowData
}
```

---

## 실험 시나리오
1. **시간 측정 비교**: 두 테스트의 실행 시간 차이 확인
2. **지연 시간 변경**: delay 값을 다르게 설정하여 차이 관찰
3. **Multiple delay**: 여러 delay가 있는 경우의 동작 확인
4. **Exception 테스트**: 예외 상황에서의 동작 차이

---

## 결론 및 참고
- **기본 원칙:**
  - 대부분의 코루틴 테스트는 `runTest` 사용
  - 실제 시간이 중요한 경우만 `runBlocking` 사용
  - 가상 시간으로 빠르고 안정적인 테스트 작성
- **성능 이점:**
  - runTest는 수천 배 빠른 실행 속도
  - CI/CD 파이프라인에서 큰 시간 절약
  - 개발자 피드백 루프 단축
- **참고 자료:**
  - [Kotlin 공식 문서: runTest](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/kotlinx.coroutines.test/run-test.html)
  - [Testing coroutines guide](https://kotlinlang.org/docs/coroutines-testing.html) 