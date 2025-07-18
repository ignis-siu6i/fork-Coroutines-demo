# B03_RunBlocking

이 파일은 runBlocking의 역할과 사용법을 설명합니다. runBlocking을 통해 메인 함수에서 코루틴을 생성하고, 동기적으로 코루틴의 완료를 기다리는 방법을 예제로 보여줍니다.

---

## Create_Coroutine_With_RunBlocking_Demo1.main
```kotlin
object Create_Coroutine_With_RunBlocking_Demo1 {
    @JvmStatic
    fun main(args: Array<String>) {
        log("Hello")
        runBlocking {
            log("Coroutine created")
            delay(1_000)
            log("Coroutine done")
        }
        log("World")
    }
}
```
- **설명:**
  - runBlocking 블록 전후로 로그를 출력하여, 코루틴이 동기적으로 동작함을 명확히 보여줍니다.
  - runBlocking 내부의 코루틴이 끝나야만 "World"가 출력됩니다.
  - **의도:** main 함수에서 코루틴을 동기적으로 실행하고, 완료를 기다리는 패턴을 실험적으로 보여줍니다.

---

## Create_Coroutine_With_RunBlocking_Demo2.main
```kotlin
object Create_Coroutine_With_RunBlocking_Demo2 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("Coroutine created")
        delay(1_000)
        log("Coroutine done")
    }
}
```
- **설명:**
  - main 함수 전체를 runBlocking으로 감싸, 코루틴이 끝날 때까지 main 함수가 종료되지 않음을 보여줍니다.
  - **의도:** runBlocking의 기본적인 사용법과, main 함수에서 코루틴을 동기적으로 실행하는 방법을 보여줍니다.

---

## 결론 및 참고
- **runBlocking의 용도:**
  - 메인 함수, 테스트 등에서 코루틴을 동기적으로 실행할 때 사용
  - 실제 앱 코드에서는 launch, async 등 비동기 빌더를 주로 사용
- **참고 자료:**
  - [Kotlin 공식 문서: runBlocking](https://kotlinlang.org/docs/coroutines-basics.html#run-blocking-coroutines) 