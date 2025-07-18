# M01_CvtCallbackToSuspendFun1

이 파일은 전통적인 콜백 기반 비동기 API를 suspend 함수로 변환하는 방법을 설명합니다. suspendCoroutine, resume/resumeWithException 사용법과 try-catch vs runCatching을 사용한 예외 처리 패턴을 보여줍니다.

---

## UsingCallback_Demo1.main
```kotlin
object UsingCallback_Demo1 {
    // Method using callback to simulate a long running task
    private fun getData(callback: AsyncCallback, status: Boolean = true) {
        // Do network request here, and then respond accordingly
        if (status) {
            callback.onSuccess("Congratulations!")
        } else {
            callback.onError(IOException("Network failure"))
        }
    }

    @JvmStatic
    fun main(args: Array<String>) {
        val callback = object : AsyncCallback {
            override fun onSuccess(result: String) {
                log("Data received: $result")
            }

            override fun onError(ex: Exception) {
                log("Caught ${ex.javaClass.simpleName}")
            }
        }

        getData(callback, true) // for success case
        getData(callback, false) // for error case
    }
}
```
- **설명:**
  - 전통적인 콜백 기반 비동기 패턴을 보여줍니다.
  - `AsyncCallback` 인터페이스로 성공과 실패를 구분하여 처리합니다.
  - 콜백 지옥(callback hell)과 에러 처리의 복잡성을 볼 수 있습니다.
  - **의도:** 콜백 기반 API의 한계와 복잡성을 실험적으로 보여줍니다.

---

## CvtToSuspendingFunction_Demo1.main
```kotlin
object CvtToSuspendingFunction_Demo1 {
    /*
     * Use `resume/resumeWithException` or `resumeWith` only
     */
    private suspend fun getData(status: Boolean = true): String = TODO()

    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        method1() // for success case
        method1(false) // for error case

        method2() // for success case
        method2(false) // for error case
    }

    suspend fun method1(status: Boolean = true) {
        try {
            getData(status).also {
                log("Data received: $it")
            }
        } catch (ex: Exception) {
            log("Caught ${ex.javaClass.simpleName}")
        }
    }

    suspend fun method2(status: Boolean = true) {
        runCatching { getData(status) }
            .onSuccess { log("Data received: $it") }
            .onFailure {
                log("Caught ${it.javaClass.simpleName}")
            }
    }
}
```
- **설명:**
  - TODO로 표시된 `getData` 함수를 suspend 함수로 구현하는 과제입니다.
  - `method1`: 전통적인 try-catch를 사용한 예외 처리
  - `method2`: `runCatching`을 사용한 함수형 스타일 예외 처리
  - **의도:** 콜백을 suspend 함수로 변환하는 실습과 다양한 예외 처리 패턴을 실험적으로 보여줍니다.

---

## 콜백을 suspend 함수로 변환하는 방법

### 1. suspendCoroutine 사용
```kotlin
private suspend fun getData(status: Boolean = true): String = suspendCoroutine { cont ->
    val callback = object : AsyncCallback {
        override fun onSuccess(result: String) {
            cont.resume(result)
        }

        override fun onError(ex: Exception) {
            cont.resumeWithException(ex)
        }
    }
    
    // 원래 콜백 함수 호출
    if (status) {
        callback.onSuccess("Congratulations!")
    } else {
        callback.onError(IOException("Network failure"))
    }
}
```

### 2. suspendCancellableCoroutine 사용 (권장)
```kotlin
private suspend fun getData(status: Boolean = true): String = suspendCancellableCoroutine { cont ->
    val callback = object : AsyncCallback {
        override fun onSuccess(result: String) {
            cont.resume(result)
        }

        override fun onError(ex: Exception) {
            cont.resumeWithException(ex)
        }
    }
    
    // 취소 처리 추가
    cont.invokeOnCancellation {
        // 진행 중인 작업 취소
    }
    
    // 실제 비동기 작업 시작
    getData(callback, status)
}
```

---

## 예외 처리 패턴 비교

### Traditional try-catch (method1)
```kotlin
suspend fun method1(status: Boolean = true) {
    try {
        val result = getData(status)
        log("Data received: $result")
    } catch (ex: Exception) {
        log("Caught ${ex.javaClass.simpleName}")
    }
}
```

### Functional style with runCatching (method2)
```kotlin
suspend fun method2(status: Boolean = true) {
    runCatching { getData(status) }
        .onSuccess { log("Data received: $it") }
        .onFailure { log("Caught ${it.javaClass.simpleName}") }
}
```

---

## 실습 과제
1. **TODO 구현**: `getData` 함수를 `suspendCoroutine`을 사용하여 구현
2. **취소 처리**: `suspendCancellableCoroutine`으로 업그레이드
3. **예외 처리 비교**: method1과 method2의 동작 차이 확인

---

## 결론 및 참고
- **마이그레이션의 이점:**
  - 콜백 지옥 해결
  - 순차적이고 읽기 쉬운 코드
  - 자동 예외 전파
  - 취소 지원
- **핵심 함수:**
  - `suspendCoroutine`: 기본 변환
  - `suspendCancellableCoroutine`: 취소 지원
  - `resume/resumeWithException`: 결과 전달
- **참고 자료:**
  - [Kotlin 공식 문서: suspendCoroutine](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/suspend-coroutine.html)
  - [Converting callback APIs](https://kotlinlang.org/docs/coroutines-basics.html#converting-callbacks) 