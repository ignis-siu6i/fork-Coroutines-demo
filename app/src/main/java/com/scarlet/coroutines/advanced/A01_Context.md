# A01_Context

이 파일은 CoroutineContext의 구성 요소(Dispatcher, Job, ExceptionHandler, Name 등)와 컨텍스트의 결합, 상속, 병합, 제거 등 다양한 활용법을 예제와 함께 설명합니다.

---

## CoroutineContext_01.main
```kotlin
object CoroutineContext_01 {
    @JvmStatic
    fun main(args: Array<String>) = runBlocking {
        log("CoroutineContext  = $coroutineContext")
        log("Name              = ${coroutineContext[CoroutineName]}")
        log("Job               = ${coroutineContext[Job]}")
        log("Dispatcher        = ${coroutineContext[ContinuationInterceptor]}")
        log("Exception handler = ${coroutineContext[CoroutineExceptionHandler]}")
    }
}
```
- **설명:**
  - runBlocking의 기본 CoroutineContext 구성 요소들을 출력하여 확인합니다.
  - Job, Dispatcher, ExceptionHandler, Name 등이 어떻게 구성되어 있는지 직접 볼 수 있습니다.
  - **의도:** CoroutineContext가 무엇으로 구성되어 있는지 실제로 보여줍니다.

---

## CoroutineContext_Creation_Plus.main
```kotlin
object CoroutineContext_Creation_Plus {
    @JvmStatic
    fun main(args: Array<String>) {
        var context: CoroutineContext = CoroutineName("My Coroutine")
        context += Dispatchers.Default
        context += Job()
    }
}
```
- **설명:**
  - + 연산자를 사용해 여러 Context 요소를 결합하는 방법을 보여줍니다.
  - **의도:** Context의 조합 방식과 빌더 패턴을 실험적으로 보여줍니다.

---

## CoroutineContext_Merge.main
```kotlin
object CoroutineContext_Merge {
    @JvmStatic
    fun main(args: Array<String>) {
        var context = CoroutineName("My Coroutine") + Dispatchers.Default + Job()
        context += CoroutineName("Your Coroutine") // 오른쪽이 왼쪽을 덮어씀
        context = context.minusKey(ContinuationInterceptor) // 특정 요소 제거
    }
}
```
- **설명:**
  - 같은 타입의 요소가 결합될 때 오른쪽이 왼쪽을 덮어쓰는 것을 보여줍니다.
  - minusKey로 특정 요소를 제거하는 방법을 보여줍니다.
  - **의도:** Context 병합 규칙과 제거 방법을 실험적으로 보여줍니다.

---

## CoroutineContext_Fold.main
```kotlin
object CoroutineContext_Fold {
    @JvmStatic
    fun main(args: Array<String>) {
        val context = CoroutineName("My Coroutine") + Dispatchers.Default + Job()
        context.fold("") { acc, elem -> "$acc : $elem" }
    }
}
```
- **설명:**
  - fold 함수로 Context의 모든 요소를 순회하며 처리하는 방법을 보여줍니다.
  - **의도:** Context의 내부 구조와 순회 방법을 실험적으로 보여줍니다.

---

## CoroutineContext_ContextInheritance_Demo.main
```kotlin
object CoroutineContext_ContextInheritance_Demo {
    @JvmStatic
    fun main(args: Array<String>) {
        runBlocking(CoroutineName("Parent Coroutine: runBlocking")) {
            launch {
                // 부모 context 상속
            }.join()
        }
    }
}
```
- **설명:**
  - 자식 코루틴이 부모의 Context를 어떻게 상속받는지 보여줍니다.
  - coroutineInfo로 실제 상속된 정보를 확인할 수 있습니다.
  - **의도:** Context 상속의 동작 방식을 실험적으로 보여줍니다.

---

## 결론 및 참고
- **CoroutineContext의 활용:**
  - 실행 환경, 취소, 예외 처리, 이름 지정 등 다양한 목적
- **참고 자료:**
  - [Kotlin 공식 문서: Coroutine Context](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html) 