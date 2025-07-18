# E03_CoroutineTest

이 파일은 Android ViewModel의 `viewModelScope`를 테스트하는 방법을 설명합니다. `Dispatchers.Main`을 교체하는 방법과 StateFlow를 사용한 UI 상태 관리를 테스트하는 실제 시나리오를 학습할 수 있습니다.

---

## 테스트 대상: ArticleViewModel
```kotlin
class ArticleViewModel() : ViewModel() {
    private val _message = MutableStateFlow("")
    val message: StateFlow<String> = _message

    fun loadMessage() {
        viewModelScope.launch {
            delay(500);
            _message.value = "Kotlin Coroutine Rocks!"
        }
    }
}
```

### ViewModel 구조 분석
- **ViewModel 상속**: Android Architecture Component 사용
- **StateFlow**: UI 상태 관리를 위한 반응형 스트림
- **viewModelScope**: ViewModel 생명주기와 연결된 코루틴 스코프
- **비동기 작업**: delay를 통한 네트워크 요청 시뮬레이션

---

## 문제가 있는 테스트
```kotlin
// How to make this test pass?
@Test
fun viewModelTest() = runTest {
    viewModel = ArticleViewModel()

    viewModel.loadMessage()
    advanceUntilIdle()

    assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
}
```

### 실패 원인
- **Dispatchers.Main 문제**: `viewModelScope`는 기본적으로 `Dispatchers.Main`을 사용
- **테스트 환경**: 테스트에서는 Main 디스패처가 설정되지 않음
- **가상 시간 미적용**: Main 디스패처가 테스트 디스패처로 교체되지 않음

---

## 해결 방법 1: Dispatchers.setMain() 사용

### 테스트 룰 없이 직접 설정
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class A03_CoroutineTest {
    private val testDispatcher = StandardTestDispatcher()
    
    @Before
    fun setup() {
        Dispatchers.setMain(testDispatcher)
    }
    
    @After
    fun tearDown() {
        Dispatchers.resetMain()
    }

    @Test
    fun viewModelTest() = runTest(testDispatcher) {
        viewModel = ArticleViewModel()

        viewModel.loadMessage()
        advanceUntilIdle()

        assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
    }
}
```

### 작동 원리
- **Main 디스패처 교체**: `Dispatchers.setMain(testDispatcher)`로 교체
- **동일한 디스패처**: `runTest`와 `viewModelScope`가 같은 디스패처 사용
- **가상 시간 적용**: `advanceUntilIdle()`이 정상 작동
- **정리**: `Dispatchers.resetMain()`으로 원상복구

---

## 해결 방법 2: CoroutineTestRule 사용 (권장)

### CoroutineTestRule 적용
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class A03_CoroutineTest {
    @get:Rule
    val coroutineTestRule = CoroutineTestRule()

    @Test
    fun viewModelTest() = runTest {
        viewModel = ArticleViewModel()

        viewModel.loadMessage()
        advanceUntilIdle()

        assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
    }
}
```

### CoroutineTestRule 구현
```kotlin
@ExperimentalCoroutinesApi
class CoroutineTestRule(
    val testDispatcher: TestDispatcher = StandardTestDispatcher()
) : TestWatcher() {

    override fun starting(description: Description) {
        super.starting(description)
        Dispatchers.setMain(testDispatcher)
    }

    override fun finished(description: Description) {
        super.finished(description)
        Dispatchers.resetMain()
    }
}
```

---

## 해결 방법 3: MainDispatcherRule 사용 (최신 권장)

### Android 공식 권장 방법
```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class A03_CoroutineTest {
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    @Test
    fun viewModelTest() = runTest {
        viewModel = ArticleViewModel()

        viewModel.loadMessage()
        advanceUntilIdle()

        assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
    }
}
```

### MainDispatcherRule 장점
- **공식 라이브러리**: kotlinx-coroutines-test에서 제공
- **자동 관리**: setUp/tearDown 자동 처리
- **간단한 사용**: 별도 구현 불필요

---

## StateFlow 테스트 패턴

### 1. 즉시 값 확인
```kotlin
@Test
fun `test immediate state change`() = runTest {
    viewModel = ArticleViewModel()
    
    // 초기값 확인
    assertThat(viewModel.message.value).isEmpty()
    
    viewModel.loadMessage()
    advanceUntilIdle()
    
    // 변경된 값 확인
    assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
}
```

### 2. Flow 수집을 통한 값 변화 추적
```kotlin
@Test
fun `test state flow changes`() = runTest {
    viewModel = ArticleViewModel()
    val states = mutableListOf<String>()
    
    // StateFlow 수집 시작
    val job = launch {
        viewModel.message.collect {
            states.add(it)
        }
    }
    
    viewModel.loadMessage()
    advanceUntilIdle()
    
    job.cancel()
    
    // 상태 변화 순서 확인
    assertThat(states).containsExactly("", "Kotlin Coroutine Rocks!")
}
```

### 3. 시간 기반 상태 변화 테스트
```kotlin
@Test
fun `test timed state changes`() = runTest {
    viewModel = ArticleViewModel()
    
    viewModel.loadMessage()
    
    // 중간 시점 확인
    advanceTimeBy(250)
    assertThat(viewModel.message.value).isEmpty()
    
    // 완료 시점 확인
    advanceTimeBy(250)
    runCurrent()
    assertThat(viewModel.message.value).isEqualTo("Kotlin Coroutine Rocks!")
}
```

---

## 실제적인 ViewModel 테스트 시나리오

### 복잡한 ViewModel 예제
```kotlin
class ArticleViewModel(
    private val repository: ArticleRepository
) : ViewModel() {
    private val _uiState = MutableStateFlow(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState

    fun loadArticles() {
        viewModelScope.launch {
            try {
                _uiState.value = UiState.Loading
                delay(500) // 로딩 시뮬레이션
                val articles = repository.getArticles()
                _uiState.value = UiState.Success(articles)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

sealed class UiState {
    object Loading : UiState()
    data class Success(val articles: List<Article>) : UiState()
    data class Error(val message: String) : UiState()
}
```

### 복잡한 시나리오 테스트
```kotlin
@Test
fun `test loading flow with success`() = runTest {
    val mockRepository = mockk<ArticleRepository>()
    coEvery { mockRepository.getArticles() } returns testArticles
    
    viewModel = ArticleViewModel(mockRepository)
    val states = mutableListOf<UiState>()
    
    val job = launch {
        viewModel.uiState.collect { states.add(it) }
    }
    
    viewModel.loadArticles()
    
    // 로딩 상태 확인
    advanceTimeBy(250)
    runCurrent()
    assertThat(states.last()).isEqualTo(UiState.Loading)
    
    // 성공 상태 확인
    advanceUntilIdle()
    assertThat(states.last()).isInstanceOf(UiState.Success::class.java)
    
    job.cancel()
}
```

---

## 실험 시나리오

### 1. Main 디스패처 문제 재현
1. **룰 없이 테스트**: Main 디스패처 설정 없이 실행
2. **에러 확인**: "Module with the Main dispatcher is missing" 에러 관찰
3. **해결책 적용**: 각 해결 방법을 차례로 적용

### 2. 시간 제어 실험
1. **delay 변경**: 500ms를 다른 값으로 변경
2. **부분 진행**: `advanceTimeBy()`로 중간 상태 확인
3. **즉시 완료**: `advanceUntilIdle()`로 즉시 완료

### 3. StateFlow 동작 확인
1. **초기값 테스트**: ViewModel 생성 직후 상태 확인
2. **변화 추적**: collect를 통한 모든 상태 변화 기록
3. **동시성 테스트**: 여러 동시 호출 시 동작 확인

---

## 모범 사례

### ✅ 권장: MainDispatcherRule 사용
```kotlin
@get:Rule
val mainDispatcherRule = MainDispatcherRule()
```

### ✅ 권장: StateFlow 수집 패턴
```kotlin
@Test
fun `test state flow`() = runTest {
    val states = mutableListOf<UiState>()
    val job = launch {
        viewModel.uiState.collect { states.add(it) }
    }
    
    // 테스트 로직
    
    job.cancel() // 정리 필수
}
```

### ✅ 권장: 의존성 주입
```kotlin
class ArticleViewModel(
    private val repository: ArticleRepository // 테스트 가능한 의존성
) : ViewModel()
```

### ❌ 피해야 할 패턴: 하드코딩된 의존성
```kotlin
class BadViewModel : ViewModel() {
    private val repository = RealRepository() // 테스트 불가능
}
```

---

## 결론 및 참고
- **핵심 학습 포인트:**
  - viewModelScope는 Dispatchers.Main을 사용함
  - 테스트에서는 Main 디스패처를 테스트 디스패처로 교체 필요
  - StateFlow를 사용한 반응형 UI 상태 관리 테스트 방법
- **Android 특화 고려사항:**
  - MainDispatcherRule 사용 권장
  - lifecycle과 연결된 코루틴 스코프 이해
  - UI 상태 관리 패턴의 테스트 중요성
- **참고 자료:**
  - [Testing Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines/test)
  - [ViewModel testing guide](https://developer.android.com/codelabs/android-room-with-a-view-kotlin#12) 