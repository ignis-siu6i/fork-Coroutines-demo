# CoroutineTestRule

이 파일은 코루틴 테스트를 위한 커스텀 JUnit 룰을 설명합니다. `TestDispatcher`를 사용하여 메인 디스패처를 교체하고, 테스트에서 가상 시간을 제어할 수 있게 해주는 헬퍼 클래스입니다.

---

## CoroutineTestRule 구현
```kotlin
@ExperimentalCoroutinesApi
class CoroutineTestRule(
    val testDispatcher: TestDispatcher = StandardTestDispatcher()
) : TestWatcher() {

    /*
     * testDispatchersProvider goes here
     */
    val testDispatcherProvider: DispatcherProvider = TODO()

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

### 구현 설명
- **TestWatcher 상속**: JUnit 테스트 생명주기에 훅을 제공합니다.
- **starting()**: 테스트 시작 전에 `Dispatchers.setMain()`으로 메인 디스패처를 테스트 디스패처로 교체합니다.
- **finished()**: 테스트 완료 후 `Dispatchers.resetMain()`으로 원래 메인 디스패처를 복원합니다.
- **testDispatcherProvider**: TODO로 표시된 부분은 실제 구현에서 채워야 합니다.

---

## TODO 완성된 구현 예제

### 1. DispatcherProvider 인터페이스
```kotlin
interface DispatcherProvider {
    val main: CoroutineDispatcher
    val io: CoroutineDispatcher
    val default: CoroutineDispatcher
    val unconfined: CoroutineDispatcher
}
```

### 2. 테스트용 DispatcherProvider 구현
```kotlin
class TestDispatcherProvider(
    private val testDispatcher: TestDispatcher
) : DispatcherProvider {
    override val main: CoroutineDispatcher = testDispatcher
    override val io: CoroutineDispatcher = testDispatcher
    override val default: CoroutineDispatcher = testDispatcher
    override val unconfined: CoroutineDispatcher = testDispatcher
}
```

### 3. 완성된 CoroutineTestRule
```kotlin
@ExperimentalCoroutinesApi
class CoroutineTestRule(
    val testDispatcher: TestDispatcher = StandardTestDispatcher()
) : TestWatcher() {

    val testDispatcherProvider: DispatcherProvider = TestDispatcherProvider(testDispatcher)

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

## 사용법 예제

### 1. 기본 사용법
```kotlin
@ExperimentalCoroutinesApi
class MyCoroutineTest {
    
    @get:Rule
    val coroutineTestRule = CoroutineTestRule()
    
    @Test
    fun `test with virtual time`() = runTest(coroutineTestRule.testDispatcher) {
        // 테스트 코드
        delay(1000)
        // 가상 시간으로 실행되어 즉시 완료
    }
}
```

### 2. ViewModel 테스트에서 사용
```kotlin
class MyViewModelTest {
    
    @get:Rule
    val coroutineTestRule = CoroutineTestRule()
    
    @Test
    fun `test viewmodel with injected dispatcher`() = runTest {
        // Given
        val viewModel = MyViewModel(coroutineTestRule.testDispatcherProvider)
        
        // When
        viewModel.performAction()
        
        // Then
        // 테스트 어설션
    }
}
```

---

## TestDispatcher 종류

### StandardTestDispatcher
```kotlin
val testRule = CoroutineTestRule(StandardTestDispatcher())
```
- **특징**: 지연된 실행 (lazy execution)
- **용도**: 가상 시간 제어가 필요한 테스트

### UnconfinedTestDispatcher
```kotlin
val testRule = CoroutineTestRule(UnconfinedTestDispatcher())
```
- **특징**: 즉시 실행 (eager execution)
- **용도**: 순서가 중요한 테스트

---

## 장점과 활용

### 장점
1. **재사용성**: 여러 테스트 클래스에서 동일한 설정 사용
2. **자동화**: 테스트 전후 설정/정리 자동화
3. **일관성**: 모든 테스트에서 동일한 디스패처 환경
4. **안전성**: 테스트 간 격리 보장

### 활용 시나리오
1. **ViewModel 테스트**: viewModelScope 테스트
2. **Repository 테스트**: withContext 사용하는 함수 테스트
3. **Service 테스트**: 백그라운드 작업 테스트
4. **통합 테스트**: 여러 컴포넌트가 함께 동작하는 테스트

---

## 실습 과제
1. **TODO 구현**: `testDispatcherProvider` 구현하기
2. **Rule 적용**: 기존 테스트에 CoroutineTestRule 적용하기
3. **비교 테스트**: StandardTestDispatcher vs UnconfinedTestDispatcher 동작 확인
4. **ViewModel 통합**: DispatcherProvider를 사용하는 ViewModel 작성

---

## 결론 및 참고
- **핵심 개념:**
  - 테스트용 디스패처로 메인 디스패처 교체
  - 가상 시간 제어를 통한 빠른 테스트
  - 테스트 간 격리와 일관성 보장
- **모범 사례:**
  - 모든 코루틴 테스트에서 TestDispatcher 사용
  - DispatcherProvider 패턴으로 의존성 주입
  - 적절한 TestDispatcher 종류 선택
- **참고 자료:**
  - [Kotlin 공식 문서: Testing coroutines](https://kotlinlang.org/docs/coroutines-testing.html)
  - [Android 가이드: Test Kotlin coroutines](https://developer.android.com/kotlin/coroutines/test) 