# A02_Dispatchers

이 파일은 코루틴 디스패처의 종류와 특성을 설명합니다. Dispatchers.Main, IO, Default, Unconfined의 차이점과 사용 시나리오, 커스텀 디스패처 작성법을 예제와 함께 보여줍니다.

---

## Dispatchers_Default_IO_Main_Demo.main
```kotlin
object Dispatchers_Default_IO_Main_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        launch(Dispatchers.Default) {
            coroutineInfo(1, "Default")
            delay(500)
        }.onCompletion("Default")

        launch(Dispatchers.IO) {
            coroutineInfo(1, "IO")
            delay(1000)
        }.onCompletion("IO")

        launch(Dispatchers.Main) {
            coroutineInfo(1, "Main")
            delay(1500)
        }.onCompletion("Main")
    }
}
```
- **설명:**
  - 세 가지 주요 디스패처의 실행 환경과 스레드 풀을 비교합니다.
  - Default: CPU 집약적 작업용 (CommonPool)
  - IO: I/O 집약적 작업용 (큰 스레드 풀)
  - Main: UI 메인 스레드용 (Android에서 주로 사용)
  - **의도:** 각 디스패처의 실제 스레드 할당과 용도를 직접 확인하게 해줍니다.

---

## Dispatchers_Unconfined_Demo.main
```kotlin
object Dispatchers_Unconfined_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        launch(Dispatchers.Unconfined) {
            log("Unconfined: 시작 thread = ${Thread.currentThread().name}")
            delay(500)
            log("Unconfined: 재개 thread = ${Thread.currentThread().name}")
        }.onCompletion("Unconfined")

        launch {
            log("Default: 시작 thread = ${Thread.currentThread().name}")
            delay(1000)
            log("Default: 재개 thread = ${Thread.currentThread().name}")
        }.onCompletion("Default")
    }
}
```
- **설명:**
  - Unconfined 디스패처의 특별한 동작을 보여줍니다.
  - 첫 번째 suspend point까지는 현재 스레드에서 실행되고, 이후에는 suspend 함수에 의해 결정된 스레드에서 재개됩니다.
  - **의도:** Unconfined의 "스레드를 고정하지 않는" 특성과 이에 따른 성능 최적화를 실험적으로 보여줍니다.

---

## Custom_Dispatcher_Demo.main
```kotlin
object Custom_Dispatcher_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val myDispatcher = newFixedThreadPoolContext(2, "MyDispatcher")
        
        try {
            launch(myDispatcher) {
                coroutineInfo(1, "Custom")
                delay(500)
            }.onCompletion("Custom")
        } finally {
            myDispatcher.close()
        }
    }
}
```
- **설명:**
  - 커스텀 디스패처를 생성하여 특정 스레드 풀을 사용하는 방법을 보여줍니다.
  - newFixedThreadPoolContext로 고정 크기 스레드 풀을 만들고 사용 후 반드시 close 해야 함을 보여줍니다.
  - **의도:** 특수한 요구사항에 맞는 커스텀 디스패처 작성법과 리소스 관리를 실험적으로 보여줍니다.

---

## 결론 및 참고
- **디스패처 선택 가이드:**
  - Default: CPU 집약적 작업 (계산, 데이터 처리)
  - IO: I/O 집약적 작업 (네트워크, 파일)
  - Main: UI 업데이트
  - Unconfined: 테스트나 특수한 경우
- **참고 자료:**
  - [Kotlin 공식 문서: Dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#dispatchers) 