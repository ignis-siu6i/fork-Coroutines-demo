# A01_Context

## 1. CoroutineContext의 구성 요소

```kotlin
log("CoroutineContext  = $coroutineContext")
log("Name              = ${coroutineContext[CoroutineName]}")
log("Job               = ${coroutineContext[Job]}")
log("Dispatcher        = ${coroutineContext[ContinuationInterceptor]}")
log("Exception handler = ${coroutineContext[CoroutineExceptionHandler]}")
```
- **설명:**
  CoroutineContext는 코루틴의 실행 환경을 정의합니다. Job, Dispatcher, ExceptionHandler, Name 등 다양한 요소를 조합할 수 있습니다.

---

## 2. Context의 결합, 병합, 제거

```kotlin
var context: CoroutineContext = CoroutineName("My Coroutine")
context += Dispatchers.Default
context += Job()
context = context.minusKey(ContinuationInterceptor)
```
- **설명:**
  여러 요소를 + 연산자로 결합하거나, minusKey로 특정 요소를 제거할 수 있습니다. 오른쪽 요소가 왼쪽을 덮어씁니다.

---

## 3. Context 상속과 EmptyContext

```kotlin
runBlocking(CoroutineName("Parent Coroutine: runBlocking")) {
    launch { /* 부모 context 상속 */ }
}
```
- **설명:**
  자식 코루틴은 부모의 context를 상속받습니다. context를 명시적으로 지정하지 않으면 부모의 dispatcher, job 등을 그대로 사용합니다.

---

## 4. 결론 및 참고

- **CoroutineContext의 활용:**
  - 실행 환경, 취소, 예외 처리, 이름 지정 등 다양한 목적

- **참고 자료:**
  - [Kotlin 공식 문서: Coroutine Context](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html) 