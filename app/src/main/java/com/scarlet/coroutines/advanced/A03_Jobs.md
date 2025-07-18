# A03_Jobs

이 파일은 코루틴의 Job에 대해 설명합니다. Job의 생명주기, 상태 변화, 부모-자식 관계, 취소와 완료 처리, SupervisorJob의 특성을 다양한 예제로 보여줍니다.

---

## Job_Demo.main
```kotlin
object Job_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = Job()
        log("job = $job")
        
        launch(job) { delay(1000) }
        delay(100)
        
        job.complete()
        log("job.complete() called")
        log("job = $job")
    }
}
```
- **설명:**
  - 독립적인 Job을 생성하고 명시적으로 complete()을 호출하는 예제입니다.
  - Job의 상태 변화(Active → Completing → Completed)를 직접 확인할 수 있습니다.
  - **의도:** Job의 기본 생명주기와 상태 관리를 실험적으로 보여줍니다.

---

## Job_hierarchy_Demo1.main
```kotlin
object Job_hierarchy_Demo1 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val request = launch {
            val job1 = launch { delay(1000) }
            val job2 = launch { delay(2000) }
            delay(500)
            log("request: 나는 두 자식을 기다린다...")
        }
        
        delay(100)
        log("Cancelling request")
        request.cancel()
        request.join()
        log("request done")
    }
}
```
- **설명:**
  - 부모 Job이 취소될 때 자식 Job들도 함께 취소되는 구조적 동시성을 보여줍니다.
  - 부모가 자식들의 완료를 기다리는 동작을 확인할 수 있습니다.
  - **의도:** 부모-자식 Job의 계층 구조와 취소 전파를 실험적으로 보여줍니다.

---

## Job_hierarchy_Demo2.main
```kotlin
object Job_hierarchy_Demo2 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val request = launch {
            repeat(3) { i ->
                launch {
                    delay((i + 1) * 200L)
                    log("코루틴 $i 완료")
                }
            }
            log("request: 모든 자식 완료 대기 중...")
        }
        
        request.join()
        log("request 완료")
    }
}
```
- **설명:**
  - 여러 자식 코루틴이 있을 때 부모가 모든 자식의 완료를 기다리는 동작을 보여줍니다.
  - **의도:** 부모 Job이 자식들의 완료를 자동으로 기다리는 구조적 동시성을 실험적으로 보여줍니다.

---

## Job_children_Demo.main
```kotlin
object Job_children_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val request = launch {
            launch { delay(1000) }
            launch { delay(2000) }
        }
        
        delay(100)
        log("request의 자식들: ${request.children.toList()}")
        request.join()
    }
}
```
- **설명:**
  - Job.children 프로퍼티로 자식 Job들에 접근하고 확인하는 방법을 보여줍니다.
  - **의도:** Job 계층 구조의 실시간 조회와 관찰을 실험적으로 보여줍니다.

---

## Job_join_vs_cancelAndJoin_Demo.main
```kotlin
object Job_join_vs_cancelAndJoin_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val job = launch {
            try {
                delay(1000)
                log("작업 완료")
            } catch (e: CancellationException) {
                log("작업 취소됨")
            }
        }
        
        delay(100)
        job.cancel()
        job.join() // vs job.cancelAndJoin()
        log("job 종료 대기 완료")
    }
}
```
- **설명:**
  - cancel() + join()과 cancelAndJoin()의 차이점을 보여줍니다.
  - 취소된 Job의 완료를 기다리는 방법들을 비교합니다.
  - **의도:** Job 취소와 완료 대기의 다양한 패턴을 실험적으로 보여줍니다.

---

## SupervisorJob_Demo.main
```kotlin
object SupervisorJob_Demo {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        val supervisor = SupervisorJob()
        
        with(CoroutineScope(coroutineContext + supervisor)) {
            val child1 = launch {
                delay(100)
                throw RuntimeException("Child1 실패!")
            }
            
            val child2 = launch {
                delay(200)
                log("Child2 완료")
            }
            
            joinAll(child1, child2)
        }
    }
}
```
- **설명:**
  - SupervisorJob의 특성을 보여줍니다. 한 자식의 실패가 다른 자식에게 전파되지 않습니다.
  - 일반 Job과 달리 독립적인 실패 처리가 가능함을 보여줍니다.
  - **의도:** SupervisorJob의 격리된 실패 처리 방식을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **Job의 핵심 개념:**
  - 생명주기 관리와 상태 추적
  - 구조적 동시성을 통한 안전한 취소
  - 부모-자식 관계를 통한 자동 정리
- **참고 자료:**
  - [Kotlin 공식 문서: Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/) 