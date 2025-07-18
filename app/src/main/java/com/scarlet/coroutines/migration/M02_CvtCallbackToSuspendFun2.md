# M02_CvtCallbackToSuspendFun2

이 파일은 실제 Retrofit API를 suspend 함수로 변환하는 실용적인 예제를 설명합니다. Retrofit Callback을 suspendCancellableCoroutine을 사용하여 suspend 함수로 변환하는 방법과 취소 처리를 보여줍니다.

---

## UsingCallback_Demo2
```kotlin
object UsingCallback_Demo2 {
    fun searchRecipes(
        query: String, api: RecipeApi, callback: RecipeCallback<List<Recipe>>
    ) {
        val call = api.search("key", query)
        call.enqueue(object : Callback<List<Recipe>> {
            override fun onResponse(call: Call<List<Recipe>>, response: Response<List<Recipe>>) {
                if (response.isSuccessful) {
                    callback.onSuccess(Resource.Success(response.body()!!))
                } else {
                    callback.onError(Resource.Error(response.message()))
                }
            }

            override fun onFailure(call: Call<List<Recipe>>, t: Throwable) {
                callback.onError(Resource.Error(t.message))
            }
        })
    }
}
```
- **설명:**
  - 실제 Retrofit API를 콜백 방식으로 사용하는 전통적인 패턴입니다.
  - `Call.enqueue()`를 사용하여 비동기 네트워크 요청을 수행합니다.
  - `onResponse`에서 성공/실패를 구분하고, `onFailure`에서 네트워크 오류를 처리합니다.
  - **의도:** 실제 네트워크 라이브러리에서 콜백 패턴의 복잡성을 실험적으로 보여줍니다.

---

## CvtToSuspendingFunction_Demo2
```kotlin
object CvtToSuspendingFunction_Demo2 {
    /*
     * TODO: Convert this method to a suspending function
     */
    fun searchRecipes(
        query: String, api: RecipeApi, callback: RecipeCallback<List<Recipe>>
    ) {
        val call = api.search("key", query)
        call.enqueue(object : Callback<List<Recipe>> {
            override fun onResponse(call: Call<List<Recipe>>, response: Response<List<Recipe>>) {
                if (response.isSuccessful) {
                    callback.onSuccess(Resource.Success(response.body()!!))
                } else {
                    callback.onError(Resource.Error(response.message()))
                }
            }

            override fun onFailure(call: Call<List<Recipe>>, t: Throwable) {
                callback.onError(Resource.Error(t.message))
            }
        })
    }
}
```
- **설명:**
  - 위의 콜백 기반 함수를 suspend 함수로 변환하는 과제입니다.
  - Retrofit의 Call을 suspendCancellableCoroutine으로 감싸야 합니다.
  - **의도:** 실제 네트워크 라이브러리 마이그레이션 실습을 실험적으로 보여줍니다.

---

## 올바른 suspend 함수 변환

### 1. 기본 변환
```kotlin
suspend fun searchRecipes(query: String, api: RecipeApi): Resource<List<Recipe>> = 
    suspendCancellableCoroutine { cont ->
        val call = api.search("key", query)
        
        call.enqueue(object : Callback<List<Recipe>> {
            override fun onResponse(call: Call<List<Recipe>>, response: Response<List<Recipe>>) {
                if (response.isSuccessful) {
                    cont.resume(Resource.Success(response.body()!!))
                } else {
                    cont.resume(Resource.Error(response.message()))
                }
            }

            override fun onFailure(call: Call<List<Recipe>>, t: Throwable) {
                cont.resume(Resource.Error(t.message))
            }
        })
        
        // 취소 처리
        cont.invokeOnCancellation {
            call.cancel()
        }
    }
```

### 2. 예외 전파 방식
```kotlin
suspend fun searchRecipesWithException(query: String, api: RecipeApi): List<Recipe> = 
    suspendCancellableCoroutine { cont ->
        val call = api.search("key", query)
        
        call.enqueue(object : Callback<List<Recipe>> {
            override fun onResponse(call: Call<List<Recipe>>, response: Response<List<Recipe>>) {
                if (response.isSuccessful) {
                    cont.resume(response.body()!!)
                } else {
                    cont.resumeWithException(IOException("HTTP ${response.code()}: ${response.message()}"))
                }
            }

            override fun onFailure(call: Call<List<Recipe>>, t: Throwable) {
                cont.resumeWithException(t)
            }
        })
        
        cont.invokeOnCancellation {
            call.cancel()
        }
    }
```

---

## 사용 방법 비교

### 콜백 방식 (기존)
```kotlin
searchRecipes("chicken", api, object : RecipeCallback<List<Recipe>> {
    override fun onSuccess(response: Resource<List<Recipe>>) {
        // 성공 처리
    }

    override fun onError(response: Resource<List<Recipe>>) {
        // 실패 처리
    }
})
```

### suspend 함수 방식 (변환 후)
```kotlin
try {
    val recipes = searchRecipesWithException("chicken", api)
    // 성공 처리
} catch (e: Exception) {
    // 실패 처리
}

// 또는 Resource 패턴
val result = searchRecipes("chicken", api)
when (result) {
    is Resource.Success -> // 성공 처리
    is Resource.Error -> // 실패 처리
}
```

---

## 취소 처리의 중요성

### 취소 없는 구현 (문제가 있음)
```kotlin
suspend fun searchRecipes(query: String, api: RecipeApi): Resource<List<Recipe>> = 
    suspendCoroutine { cont ->
        val call = api.search("key", query)
        call.enqueue(callback) // 취소되어도 네트워크 요청 계속 진행
    }
```

### 취소 지원 구현 (권장)
```kotlin
suspend fun searchRecipes(query: String, api: RecipeApi): Resource<List<Recipe>> = 
    suspendCancellableCoroutine { cont ->
        val call = api.search("key", query)
        call.enqueue(callback)
        
        cont.invokeOnCancellation {
            call.cancel() // 네트워크 요청도 함께 취소
        }
    }
```

---

## 실습 과제
1. **TODO 구현**: `searchRecipes` 함수를 suspend 함수로 변환
2. **취소 처리**: `invokeOnCancellation`으로 Call 취소 구현
3. **예외 처리**: Resource 패턴과 예외 전파 방식 비교
4. **실제 사용**: 변환된 함수를 코루틴에서 호출하여 테스트

---

## 결론 및 참고
- **Retrofit 마이그레이션 이점:**
  - 동기식 코드 스타일
  - 자동 취소 지원
  - 예외 처리 간소화
  - 구조적 동시성 지원
- **핵심 포인트:**
  - `suspendCancellableCoroutine` 사용
  - `invokeOnCancellation`으로 리소스 정리
  - 적절한 예외 처리 선택
- **참고 자료:**
  - [Retrofit 공식 문서: Kotlin Coroutines](https://square.github.io/retrofit/kotlinx.html)
  - [Android 공식 가이드: Retrofit with Coroutines](https://developer.android.com/kotlin/coroutines/coroutines-adv) 