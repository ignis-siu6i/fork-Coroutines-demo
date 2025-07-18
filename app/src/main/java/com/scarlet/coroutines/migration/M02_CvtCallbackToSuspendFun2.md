# M02_CvtCallbackToSuspendFun2

## 1. Retrofit 콜백 기반 API

```kotlin
fun searchRecipes(query: String, api: RecipeApi, callback: RecipeCallback<List<Recipe>>) {
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
```
- **설명:**
  Retrofit 등 네트워크 라이브러리는 콜백 기반 비동기 API를 제공합니다. 콜백 패턴은 에러 처리, 흐름 제어가 복잡해질 수 있습니다.

---

## 2. suspendCoroutine을 이용한 변환

```kotlin
suspend fun searchRecipesSuspend(query: String, api: RecipeApi): Resource<List<Recipe>> = suspendCoroutine { cont ->
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
}
```
- **설명:**
  suspendCoroutine을 사용해 Retrofit 콜백 기반 API를 suspend 함수로 변환하면, 코루틴 내에서 네트워크 요청을 동기식 코드처럼 사용할 수 있습니다.

---

## 3. 결론 및 참고

- **콜백 → suspend 변환의 장점:**
  - 네트워크 요청, 에러 처리, 흐름 제어가 간결해짐

- **참고 자료:**
  - [공식 문서: Retrofit + Coroutines](https://square.github.io/retrofit/)
  - [Kotlin 공식 문서: suspendCoroutine](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.jvm.internal/-suspend-coroutine/) 