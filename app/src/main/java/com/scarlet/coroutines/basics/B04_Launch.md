# B04_Launch

이 파일은 launch 빌더를 사용한 코루틴 생성과 실행, join을 통한 동기화, GlobalScope 사용의 위험성, CoroutineScope의 개념을 다양한 예제로 설명합니다. 각 main 함수는 launch의 다양한 사용법과 실무에서 주의할 점을 실험적으로 보여줍니다.

---

## Launch_Demo1.main
```kotlin
object Launch_Demo1 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("1. before launch")
        launch {
            log("3. before save")
            save(User("A001", "Jody", 33))
            log("4. after save")
        }
        log("2. after launch")
    }
}
```
- **설명:**
  - runBlocking 내에서 launch로 코루틴을 생성합니다. launch 블록은 비동기로 실행되며, main 함수의 흐름과 별개로 동작합니다.
  - 로그 순서를 통해 launch가 비동기로 동작함을 확인할 수 있습니다.
  - **의도:** launch의 기본 동작(비동기 실행, 부모 runBlocking이 끝날 때까지 대기)을 보여줍니다.

---

## Launch_Demo2.main
```kotlin
object Launch_Demo2 {
    @JvmStatic
    fun main(args: Array<String>) {
        log("X. Start")
        runBlocking {
            launch {
                delay(1_000)
                log("X. child 1 done.")
            }
            launch {
                delay(2_000)
                log("X. child 2 done.")
            }
            log("X. end of runBlocking")
        }
        log("X. Done")
    }
}
```
- **설명:**
  - 여러 launch를 동시에 실행하고, runBlocking이 끝날 때까지 모든 자식 코루틴이 완료될 때까지 대기합니다.
  - **의도:** 여러 코루틴의 동시 실행과 부모 runBlocking의 동기적 대기 구조를 보여줍니다.

---

## Launch_Join_Demo.main
```kotlin
object Launch_Join_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("1. start of runBlocking")
        launch {
            log("2. child 1 start")
            delay(1_000)
            log("3. child 1 done")
        }
        log("4. Done")
    }
}
```
- **설명:**
  - launch로 생성한 자식 코루틴이 완료되기 전에 부모 runBlocking의 마지막 로그가 먼저 출력됩니다.
  - **의도:** launch는 기본적으로 비동기 실행이므로, 명시적으로 join하지 않으면 부모가 먼저 종료될 수 있음을 보여줍니다.

---

## GlobalScope_Demo.main
```kotlin
object GlobalScope_Demo {
    @OptIn(DelicateCoroutinesApi::class)
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("1. start of runBlocking")
        GlobalScope.launch {
            log("2. before save")
            save(User("A001", "Jody", 33))
            log("3. after save")
        }//.join()
        log("4. Done.")
    }
}
```
- **설명:**
  - GlobalScope.launch는 앱 전체 생명주기와 무관하게 동작합니다. runBlocking이 끝나도 GlobalScope의 코루틴은 계속 실행될 수 있습니다.
  - **의도:** GlobalScope 사용의 위험성(메모리 누수, 예측 불가한 동작)을 보여줍니다.

---

## CoroutineScope_Sneak_Preview_Demo.main
```kotlin
object CoroutineScope_Sneak_Preview_Demo {
    @JvmStatic
    fun main(args: Array<String>) {
        val scope = CoroutineScope(Job())
        val job = scope.launch {
            log("1. before save")
            save(User("A001", "Jody", 33))
            log("2. after save")
        }
        // force the main thread wait
//        Thread.sleep(2_000)
//        runBlocking { job.join() }
        log("3. Done.")
    }
}
```
- **설명:**
  - 명시적으로 CoroutineScope를 생성해 launch를 실행합니다. main 스레드가 job의 완료를 기다리지 않으면, 자식 코루틴이 끝나기 전에 main 함수가 종료될 수 있습니다.
  - **의도:** CoroutineScope와 main 스레드의 생명주기 분리를 보여줍니다.

---

## 결론 및 참고
- **launch의 용도:**
  - 비동기 작업, 구조적 동시성 구현에 필수
  - scope와 함께 사용해 생명주기 관리
- **참고 자료:**
  - [Kotlin 공식 문서: launch](https://kotlinlang.org/docs/coroutines-basics.html#launching-a-coroutine) 