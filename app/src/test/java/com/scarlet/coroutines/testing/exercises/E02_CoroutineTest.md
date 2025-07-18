# E02_CoroutineTest

이 파일은 `Dispatchers.IO`를 사용하는 Repository 패턴의 테스트 방법을 설명합니다. 실제 디스패처를 테스트 디스패처로 교체하는 방법과 의존성 주입을 통한 테스트 가능한 설계를 학습할 수 있습니다.

---

## 테스트 대상 클래스들
```kotlin
interface ApiService {
    suspend fun populate()
    suspend fun getArticles(): List<Article>
}

class FakeApiService : ApiService {
    private val articles = mutableListOf<Article>()

    override suspend fun populate() {
        delay(1_000)
        articles.add(Article("1", "Title 1", "Body 1"))
        articles.add(Article("2", "Title 2", "Body 2"))
        articles.add(Article("3", "Title 3", "Body 3"))
    }

    override suspend fun getArticles(): List<Article> {
        delay(500)
        return articles
    }
}
```

### 클래스 설명
- **ApiService**: 데이터 초기화와 조회를 위한 인터페이스
- **FakeApiService**: 테스트용 구현체 (지연 시간 포함)
- **delay 사용**: 실제 네트워크 지연을 시뮬레이션

---

## 문제가 있는 Repository 구현
```kotlin
class Repository(
    private val apiService: ApiService,
    // TODO() - Add a coroutine dispatcher
) {
    private val scope = CoroutineScope(Dispatchers.IO)

    fun initialize() {
        scope.launch(Dispatchers.IO) {
            apiService.populate();
        }
    }

    suspend fun loadData() = withContext(Dispatchers.IO) {
        apiService.getArticles()
    }
}
```

### 문제점 분석
- **하드코딩된 디스패처**: `Dispatchers.IO`가 직접 사용됨
- **테스트 불가능**: 테스트에서 가상 시간 제어 불가
- **의존성 주입 누락**: TODO 주석으로 표시된 미완성 부분

---

## 실패하는 테스트
```kotlin
// How to make this test pass?
@Test
fun repositoryTest() = runTest {
    repository = Repository(FakeApiService())
    repository.initialize()

    advanceUntilIdle()

    val articles = repository.loadData()
    assertThat(articles).containsExactly(
        Article("1", "Title 1", "Body 1"),
        Article("2", "Title 2", "Body 2"),
        Article("3", "Title 3", "Body 3")
    )
}
```

### 실패 원인
- **다른 디스패처**: `Dispatchers.IO`와 테스트 디스패처가 분리됨
- **가상 시간 미적용**: 실제 1000ms + 500ms 지연 발생
- **동기화 문제**: `advanceUntilIdle()`이 실제 IO 작업에 영향 없음

---

## 해결 방법 1: DispatcherProvider 패턴

### 1. DispatcherProvider 인터페이스 생성
```kotlin
interface DispatcherProvider {
    val main: CoroutineDispatcher
    val io: CoroutineDispatcher
    val default: CoroutineDispatcher
}

class DefaultDispatcherProvider : DispatcherProvider {
    override val main = Dispatchers.Main
    override val io = Dispatchers.IO
    override val default = Dispatchers.Default
}

class TestDispatcherProvider(
    private val testDispatcher: TestDispatcher
) : DispatcherProvider {
    override val main = testDispatcher
    override val io = testDispatcher
    override val default = testDispatcher
}
```

### 2. Repository 수정
```kotlin
class Repository(
    private val apiService: ApiService,
    private val dispatcherProvider: DispatcherProvider
) {
    private val scope = CoroutineScope(dispatcherProvider.io)

    fun initialize() {
        scope.launch(dispatcherProvider.io) {
            apiService.populate()
        }
    }

    suspend fun loadData() = withContext(dispatcherProvider.io) {
        apiService.getArticles()
    }
}
```

### 3. 수정된 테스트
```kotlin
@Test
fun repositoryTest() = runTest {
    val testDispatcherProvider = TestDispatcherProvider(
        testDispatcher = this.coroutineContext[TestDispatcher]!!
    )
    
    repository = Repository(FakeApiService(), testDispatcherProvider)
    repository.initialize()

    advanceUntilIdle()

    val articles = repository.loadData()
    assertThat(articles).containsExactly(
        Article("1", "Title 1", "Body 1"),
        Article("2", "Title 2", "Body 2"),
        Article("3", "Title 3", "Body 3")
    )
}
```

---

## 해결 방법 2: TestScope.backgroundScope 사용

### 간단한 접근법
```kotlin
@Test
fun repositoryTestWithBackgroundScope() = runTest {
    // TestScope의 backgroundScope 사용
    val repository = Repository(
        apiService = FakeApiService(),
        scope = backgroundScope  // 테스트 스코프와 연결됨
    )
    
    repository.initialize()
    advanceUntilIdle()
    
    val articles = repository.loadData()
    assertThat(articles).hasSize(3)
}
```

---

## 해결 방법 3: 생성자 주입으로 간단화

### Repository 수정 (권장)
```kotlin
class Repository(
    private val apiService: ApiService,
    private val dispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    private val scope = CoroutineScope(dispatcher)

    fun initialize() {
        scope.launch {
            apiService.populate()
        }
    }

    suspend fun loadData() = withContext(dispatcher) {
        apiService.getArticles()
    }
}
```

### 테스트 코드
```kotlin
@Test
fun repositoryTest() = runTest {
    repository = Repository(
        apiService = FakeApiService(),
        dispatcher = testScheduler  // 또는 Dispatchers.Unconfined
    )
    
    repository.initialize()
    advanceUntilIdle()

    val articles = repository.loadData()
    assertThat(articles).containsExactly(
        Article("1", "Title 1", "Body 1"),
        Article("2", "Title 2", "Body 2"),
        Article("3", "Title 3", "Body 3")
    )
}
```

---

## 각 해결 방법 비교

### 방법 1: DispatcherProvider (복합 앱에 적합)
**장점:**
- 모든 디스패처를 중앙 관리
- 의존성 주입 프레임워크와 잘 통합
- 확장성 좋음

**단점:**
- 초기 설정 복잡
- 작은 프로젝트에는 과도함

### 방법 2: backgroundScope (간단한 케이스)
**장점:**
- 최소한의 변경
- 빠른 적용 가능

**단점:**
- 제한적인 제어
- 복잡한 시나리오에 부적합

### 방법 3: 생성자 주입 (권장)
**장점:**
- 간단하고 명확
- 테스트하기 쉬움
- 적당한 복잡도

**단점:**
- 다중 디스패처 사용시 파라미터 증가

---

## 실험 시나리오

### 1. 문제 재현
1. **원본 테스트 실행**: 실패하는 이유 확인
2. **시간 측정**: 실제 지연 시간 관찰
3. **로그 추가**: 어떤 스레드에서 실행되는지 확인

### 2. 해결 방법 적용
1. **각 방법 시도**: 세 가지 해결 방법 모두 적용
2. **성능 비교**: 가상 시간 vs 실제 시간 차이
3. **복잡도 증가**: 더 많은 비동기 작업 추가

### 3. 실제 앱 시나리오
```kotlin
// 실제 프로덕션 코드
class ArticleRepository(
    private val apiService: ApiService,
    private val dispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun refreshAndGet(): List<Article> = withContext(dispatcher) {
        apiService.populate()
        apiService.getArticles()
    }
}
```

---

## 모범 사례

### ✅ 권장: 디스패처 주입
```kotlin
class MyRepository(
    private val apiService: ApiService,
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun loadData() = withContext(ioDispatcher) {
        apiService.getData()
    }
}
```

### ✅ 권장: DispatcherProvider 사용 (대규모 앱)
```kotlin
class MyRepository @Inject constructor(
    private val apiService: ApiService,
    private val dispatchers: DispatcherProvider
) {
    suspend fun loadData() = withContext(dispatchers.io) {
        apiService.getData()
    }
}
```

### ❌ 피해야 할 패턴: 하드코딩
```kotlin
class MyRepository {
    suspend fun loadData() = withContext(Dispatchers.IO) { // 테스트 불가능
        apiService.getData()
    }
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - 하드코딩된 디스패처의 문제점
  - 의존성 주입을 통한 테스트 가능한 설계
  - 다양한 디스패처 주입 패턴
- **설계 원칙:**
  - 테스트 가능성을 고려한 설계
  - 의존성 역전 원칙 적용
  - 적절한 추상화 수준 선택
- **참고 자료:**
  - [Testing with TestDispatchers](https://kotlinlang.org/docs/coroutines-testing.html#testing-with-testdispatchers)
  - [Dependency Injection patterns](https://developer.android.com/training/dependency-injection) 