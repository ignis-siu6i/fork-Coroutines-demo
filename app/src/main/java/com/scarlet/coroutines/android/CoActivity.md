# CoActivity

이 파일은 Android에서 lifecycle-aware 코루틴 사용법을 설명합니다. lifecycleScope vs lifecycle.coroutineScope의 차이점, repeatOnLifecycle의 올바른 사용법, 그리고 deprecated된 메서드들과의 비교를 보여줍니다.

---

## onCreate - lifecycleScope vs lifecycle.coroutineScope
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    Log.e(TAG, "[onCreate] launching started ...")
    lifecycle.coroutineScope.launch {
        Log.e(TAG, "launch started")
        val recipes = apiService.getRecipes()
        Log.e(TAG, "recipes in launch = $recipes")
    }.invokeOnCompletion {
        Log.e(TAG, "launch completed: $it")
    }
}
```
- **설명:**
  - `lifecycle.coroutineScope`를 사용하여 생명주기와 연결된 코루틴을 시작합니다.
  - `lifecycleScope`와 `lifecycle.coroutineScope`는 동일한 스코프입니다.
  - onCreate에서 시작된 코루틴은 액티비티가 파괴될 때까지 실행됩니다.
  - **의도:** Android 생명주기와 연결된 기본적인 코루틴 사용법을 실험적으로 보여줍니다.

---

## repeatOnLifecycle 사용법 (주석 처리된 코드)
```kotlin
lifecycleScope.launch {
    Log.e(TAG, "repeatOnLifecycle launched, job = ${coroutineContext[Job]}")
    lifecycle.repeatOnLifecycle(Lifecycle.State.RESUMED) {
        Log.e(TAG, "${spaces(4)}repeatOnLifeCycle at RESUMED started, job = ${coroutineContext[Job]}")
        val recipes = apiService.getRecipes()
        Log.e(TAG, "${spaces(4)}recipes in repeatOnLifeCycle = $recipes")
    }
    Log.e(TAG, "See when i am printed ...")
}.invokeOnCompletion {
    Log.e(TAG, "launch for repeatOnLifeCycle completed: $it")
}
```
- **설명:**
  - `repeatOnLifecycle(Lifecycle.State.RESUMED)`는 RESUMED 상태에서만 실행됩니다.
  - 액티비티가 PAUSED 상태가 되면 일시 중단되고, RESUMED 상태가 되면 다시 시작됩니다.
  - deprecated된 `launchWhenResumed` 등을 대체하는 권장 방법입니다.
  - **의도:** 생명주기 상태에 따른 코루틴 제어와 메모리 효율성을 실험적으로 보여줍니다.

---

## FakeRemoteDataSource - 네트워크 시뮬레이션
```kotlin
private class FakeRemoteDataSource {
    suspend fun getRecipes(): Resource<List<Recipe>> {
        return withContext(Dispatchers.IO) {
            delay(FAKE_NETWORK_DELAY)
            Resource.Success(mRecipes.values.toList())
        }
    }
    
    companion object {
        var FAKE_NETWORK_DELAY = 0L
    }
}
```
- **설명:**
  - `withContext(Dispatchers.IO)`로 네트워크 작업을 시뮬레이션합니다.
  - `delay(FAKE_NETWORK_DELAY)`로 네트워크 지연을 모방합니다.
  - 실제 앱에서 Repository 패턴과 유사한 구조입니다.
  - **의도:** Android에서 네트워크 작업과 코루틴의 올바른 결합을 실험적으로 보여줍니다.

---

## 생명주기 콜백과 로깅
```kotlin
override fun onStart() {
    super.onStart()
    Log.e(TAG, "[onStart]")
}

override fun onStop() {
    super.onStop()
    Log.e(TAG, "[onStop]")
}

override fun onPause() {
    super.onPause()
    Log.e(TAG, "[onPause]")
}

override fun onResume() {
    super.onResume()
    Log.e(TAG, "[onResume]")
}

override fun onDestroy() {
    super.onDestroy()
    Log.e(TAG, "[onDestroy]")
}
```
- **설명:**
  - 각 생명주기 콜백에서 로그를 출력하여 코루틴 실행과 생명주기의 관계를 추적할 수 있습니다.
  - repeatOnLifecycle과 함께 사용하면 어떤 상태에서 코루틴이 실행/중단되는지 확인 가능합니다.
  - **의도:** 생명주기와 코루틴의 실행 패턴을 시각화하고 이해를 도와줍니다.

---

## 실험 시나리오
1. **기본 실행**: 앱 시작 시 코루틴이 언제 시작되고 완료되는지 로그 확인
2. **repeatOnLifecycle 테스트**: 주석을 해제하고 앱을 백그라운드/포그라운드로 전환하며 동작 확인
3. **생명주기 추적**: 다양한 생명주기 전환 시 코루틴 상태 변화 관찰

---

## Deprecated vs Recommended
- **Deprecated**: `launchWhenCreated`, `launchWhenStarted`, `launchWhenResumed`
- **Recommended**: `lifecycle.repeatOnLifecycle(Lifecycle.State.XXX)`

---

## 결론 및 참고
- **Android 생명주기 인식 코루틴:**
  - lifecycleScope로 생명주기와 연결
  - repeatOnLifecycle로 특정 상태에서만 실행
  - 자동 취소로 메모리 누수 방지
  - 백그라운드에서 불필요한 작업 중단
- **참고 자료:**
  - [Android 공식 문서: Use lifecycle-aware coroutines](https://developer.android.com/topic/libraries/architecture/coroutines)
  - [repeatOnLifecycle API](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary#(androidx.lifecycle.Lifecycle).repeatOnLifecycle(androidx.lifecycle.Lifecycle.State,kotlin.coroutines.SuspendFunction1)) 