# B03_RunBlocking

## 1. runBlocking을 이용한 코루틴 생성

```kotlin
runBlocking {
    log("Coroutine created")
    delay(1_000)
    log("Coroutine done")
}
```
- **설명:**
  runBlocking은 메인 함수나 테스트 코드에서 코루틴을 동기적으로 실행할 때 사용합니다. 내부 블록이 끝날 때까지 현재 스레드를 블로킹합니다.

---

## 2. runBlocking의 위치에 따른 동작 차이

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
  runBlocking 블록 전후로 코드가 실행되는 순서를 통해, 코루틴이 동기적으로 동작함을 확인할 수 있습니다.

---

## 3. 결론 및 참고

- **runBlocking의 용도:**
  - 메인 함수, 테스트 등에서 코루틴을 동기적으로 실행할 때 사용
  - 실제 앱 코드에서는 launch, async 등 비동기 빌더를 주로 사용

- **참고 자료:**
  - [Kotlin 공식 문서: runBlocking](https://kotlinlang.org/docs/coroutines-basics.html#run-blocking-coroutines) 