# CoActivity

## 1. lifecycleScope, coroutineScope, repeatOnLifecycle의 활용

```kotlin
lifecycle.coroutineScope.launch {
    val recipes = apiService.getRecipes()
    Log.e(TAG, "recipes in launch = $recipes")
}

lifecycleScope.launch {
    lifecycle.repeatOnLifecycle(Lifecycle.State.RESUMED) {
        val recipes = apiService.getRecipes()
        Log.e(TAG, "recipes in repeatOnLifeCycle = $recipes")
    }
}
```
- **설명:**
  lifecycleScope, coroutineScope, repeatOnLifecycle을 활용해 Activity/Fragment의 생명주기에 맞춰 안전하게 코루틴을 실행할 수 있습니다. 네트워크 데이터 로딩, UI 업데이트, 생명주기 관리에 적합합니다.

---

## 2. FakeRemoteDataSource와 비동기 데이터 처리

```kotlin
private class FakeRemoteDataSource {
    suspend fun getRecipes(): Resource<List<Recipe>> {
        return withContext(Dispatchers.IO) {
            delay(FAKE_NETWORK_DELAY)
            Resource.Success(mRecipes.values.toList())
        }
    }
}
```
- **설명:**
  실제 네트워크 대신 가짜 데이터 소스를 사용해 비동기 데이터 로딩을 연습할 수 있습니다. withContext(Dispatchers.IO)로 IO 작업을 안전하게 분리합니다.

---

## 3. 결론 및 참고

- **Android에서 코루틴의 활용:**
  - 생명주기 안전, 네트워크/비동기 데이터 처리, UI 업데이트 등

- **참고 자료:**
  - [공식 문서: lifecycleScope](https://developer.android.com/topic/libraries/architecture/coroutines#lifecyclescope)
  - [공식 문서: repeatOnLifecycle](https://developer.android.com/topic/libraries/architecture/coroutines#repeatonlifecycle) 