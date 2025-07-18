# M01_CvtCallbackToSuspendFun1

## 1. 콜백 기반 비동기 코드

```kotlin
private fun getData(callback: AsyncCallback, status: Boolean = true) {
    if (status) {
        callback.onSuccess("Congratulations!")
    } else {
        callback.onError(IOException("Network failure"))
    }
}
```
- **설명:**
  전통적인 콜백 패턴은 비동기 작업의 결과를 콜백 인터페이스로 전달합니다. 콜백 지옥, 에러 처리의 어려움 등 한계가 있습니다.

---

## 2. suspendCoroutine을 이용한 변환

```kotlin
private suspend fun getData(status: Boolean = true): String = suspendCoroutine { cont ->
    getData(object : AsyncCallback {
        override fun onSuccess(result: String) = cont.resume(result)
        override fun onError(ex: Exception) = cont.resumeWithException(ex)
    }, status)
}
```
- **설명:**
  suspendCoroutine을 사용하면 콜백 기반 비동기 코드를 코루틴의 suspend 함수로 변환할 수 있습니다. resume, resumeWithException으로 결과/에러를 자연스럽게 처리합니다.

---

## 3. 결론 및 참고

- **콜백 → suspend 변환의 장점:**
  - 동기식 코드처럼 읽기 쉬운 비동기 처리
  - 예외 처리, 흐름 제어가 간결해짐

- **참고 자료:**
  - [Kotlin 공식 문서: suspendCoroutine](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.jvm.internal/-suspend-coroutine/) 