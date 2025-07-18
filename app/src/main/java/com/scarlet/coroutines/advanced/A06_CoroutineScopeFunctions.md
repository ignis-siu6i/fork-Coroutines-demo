# A06_CoroutineScopeFunctions

이 파일은 코루틴 스코프 함수들(coroutineScope, supervisorScope, withContext, withTimeout 등)의 특성과 사용법을 설명합니다. 각 스코프 함수의 실행 방식, Context 상속, 취소 전파, 예외 처리 등을 다양한 예제로 보여줍니다.

---

## coroutineScope_Demo1.main
```kotlin
object coroutineScope_Demo1 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("runBlocking: $coroutineContext")

        val a = coroutineScope {
            log("a: $coroutineContext")
            delay(1_000)
            10
        }
        log("a is calculated")
        val b = coroutineScope {
            log("b: $coroutineContext")
            delay(1_000)
            20
        }
        log("a = $a, b = $b")
    }
}
```
- **설명:**
  - coroutineScope는 새로운 스코프를 만들지만 현재 코루틴을 일시 중단하고 순차적으로 실행됩니다.
  - 부모의 CoroutineContext를 상속받지만 새로운 Job을 생성합니다.
  - **의도:** coroutineScope의 in-place 실행과 Context 상속을 실험적으로 보여줍니다.

---

## coroutineScope_Demo2.main
```kotlin
object coroutineScope_Demo2 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("runBlocking begins")

        coroutineScope {
            log("Launching children ...")

            launch {
                log("child1 starts")
                delay(2_000)
            }.onCompletion("child1")

            launch {
                log("child2 starts")
                delay(1_000)
            }.onCompletion("child2")

            delay(10)

            log("Waiting until children are completed ...")
        }

        log("Done!")
    }
}
```
- **설명:**
  - coroutineScope 내에서 생성된 자식 코루틴들은 모두 완료될 때까지 기다립니다.
  - 구조적 동시성을 보장하여 자식들이 완료되기 전에는 스코프가 종료되지 않습니다.
  - **의도:** coroutineScope의 구조적 동시성과 자식 코루틴 관리를 실험적으로 보여줍니다.

---

## coroutineScope_Demo3.main
```kotlin
object coroutineScope_Demo3 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("runBlocking begins")

        try {
            coroutineScope {
                log("Launching children ...")

                launch {
                    log("child1 starts")
                    delay(2_000)
                }.onCompletion("child1")

                launch {
                    log("child2 starts")
                    delay(1_000)
                    throw RuntimeException("Oops")
                }.onCompletion("child2")

                delay(10)

                log("Waiting until children are completed ...")
            }
        } catch (ex: Exception) {
            log("Caught exception: ${ex.javaClass.simpleName}")
        }

        log("Done!")
    }
}
```
- **설명:**
  - coroutineScope 내에서 자식 코루틴의 예외가 발생하면 다른 자식들도 취소됩니다.
  - 예외는 스코프 밖으로 전파되어 try-catch로 잡을 수 있습니다.
  - **의도:** coroutineScope의 예외 전파와 자식 코루틴 취소를 실험적으로 보여줍니다.

---

## What_We_Want.main vs Not_What_We_Want.main
```kotlin
object What_We_Want {
    private suspend fun getUserDetails(): Details = coroutineScope {
        val userName = async { getUserName() }
        val followersNumber = async { getFollowersNumber() }

        Details(userName.await(), followersNumber.await())
    }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val details = try {
            getUserDetails()
        } catch (e: ApiException) {
            log("Error: ${e.code}")
            null
        }
        val tweets = async { getTweets() }
        log("User: $details")
        log("Tweets: ${tweets.await()}")
    }
}
```
- **설명:**
  - coroutineScope를 사용하면 함수 내부의 병렬 작업이 실패해도 외부 코루틴에 영향을 주지 않습니다.
  - 스코프 함수를 사용하지 않고 부모 스코프를 직접 전달하면 예외 격리가 되지 않습니다.
  - **의도:** coroutineScope의 올바른 사용법과 예외 격리 효과를 실험적으로 보여줍니다.

---

## withContext_Demo.main
```kotlin
object withContext_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val parent = launch(CoroutineName("parent")) {
            coroutineInfo(0)

            coroutineScope {
                coroutineContext.job.onCompletion("coroutineScope")
                log("\t\tInside coroutineScope")
                coroutineInfo(1)
                delay(100)
            }

            withContext(CoroutineName("child 1") + Dispatchers.Default) {
                coroutineContext.job.onCompletion("withContext")
                log("\t\tInside first withContext")
                coroutineInfo(1)
                delay(500)
            }

            Executors.newFixedThreadPoolContext(3, "MyDispatcher").use { ctx ->
                withContext(CoroutineName("child 2") + ctx) {
                    coroutineContext.job.onCompletion("newFixedThreadPool")
                    log("\t\tInside second withContext")
                    coroutineInfo(1)
                    delay(1_000)
                }
            }
        }.onCompletion("parent")

        delay(50)
        log("children after 50ms  = ${parent.children.toList()}")
        delay(200)
        log("children after 250ms = ${parent.children.toList()}")
        delay(600)
        log("children after 850ms = ${parent.children.toList()}")
        parent.join()
    }
}
```
- **설명:**
  - withContext는 Context를 변경하여 실행하는 스코프 함수입니다.
  - Dispatcher를 변경하거나 이름을 추가하는 등 Context 수정이 가능합니다.
  - **의도:** withContext의 Context 변경 능력과 실행 스레드 전환을 실험적으로 보여줍니다.

---

## MainSafety_Demo.main
```kotlin
object MainSafety_Demo {
    private suspend fun fibonacci(n: Long): Long =
        withContext(Dispatchers.Default) {
            log(coroutineContext)
            fib(n)
        }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking(CoroutineName("parent") + Dispatchers.Swing) {
        log(coroutineContext)

        val job1 = launch {
            log("fib(44) = ${fibonacci(44)}")
        }

        val job2 = launch {
            for (i in 1..10) {
                log("i = $i")
                delay(500)
            }
        }

        joinAll(job1, job2)
        log("Done")
    }
}
```
- **설명:**
  - withContext(Dispatchers.Default)로 CPU 집약적 작업을 별도 스레드에서 실행합니다.
  - Main 스레드에서 실행되는 다른 작업들을 블로킹하지 않습니다.
  - **의도:** withContext를 통한 메인 스레드 안전성과 작업 분리를 실험적으로 보여줍니다.

---

## Timeout.main
```kotlin
object Timeout {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking<Unit> {
        launch {
            launch { // will be cancelled by its parent
                delay(2_000)
                log("Will not be printed")
            }.onCompletion("grandchild")
            withTimeout(1_000) { // we cancel launch
                delay(1_500)
            }
        }.onCompletion("child 1")

        launch {
            delay(2_000)
            log("child2 done")
        }.onCompletion("child 2")
    }
}
```
- **설명:**
  - withTimeout은 지정된 시간 내에 실행되지 않으면 TimeoutCancellationException을 던집니다.
  - 타임아웃으로 인한 취소는 자식 코루틴까지 전파됩니다.
  - **의도:** withTimeout의 시간 제한과 취소 전파를 실험적으로 보여줍니다.

---

## WithTimeoutOrNull_Demo.main
```kotlin
object WithTimeoutOrNull_Demo {
    private suspend fun fetchUser(): User {
        // Runs forever
        while (true) {
            yield()
        }
    }

    private suspend fun getUserOrNull(): User? =
        withTimeoutOrNull(3_000) {
            fetchUser()
        }

    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val user = getUserOrNull()
        log("User: $user")
    }
}
```
- **설명:**
  - withTimeoutOrNull은 타임아웃 시 예외 대신 null을 반환합니다.
  - 타임아웃을 예외가 아닌 정상적인 결과로 처리할 때 유용합니다.
  - **의도:** withTimeoutOrNull의 안전한 타임아웃 처리를 실험적으로 보여줍니다.

---

## 결론 및 참고
- **스코프 함수 선택 가이드:**
  - coroutineScope: 구조적 동시성과 예외 전파
  - supervisorScope: 독립적인 자식 실행
  - withContext: Context 변경
  - withTimeout/withTimeoutOrNull: 시간 제한
- **참고 자료:**
  - [Kotlin 공식 문서: Coroutine Scope Functions](https://kotlinlang.org/docs/coroutines-basics.html#scope-functions) 