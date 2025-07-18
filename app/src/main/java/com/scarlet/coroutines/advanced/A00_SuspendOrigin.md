# A00_SuspendOrigin

이 파일은 코루틴의 suspend 함수가 어떻게 동작하는지, 그리고 일반 함수와의 차이점을 설명합니다. Thread.sleep과 delay의 차이, suspend 키워드의 의미와 동작 원리를 간단한 예제로 보여줍니다.

---

## SuspendOrigin.main
```kotlin
object SuspendOrigin {
    @JvmStatic
    fun main(args: Array<String>) {
        log("main started")
        log("result = ${fooWithDelay(3, 4)}")
        log("main end")
    }
    
    private fun fooWithDelay(a: Int, b: Int): Int {
        log("step 1")
        Thread.sleep(3_000)
        log("step 2")
        return a + b
    }
}
```
- **설명:**
  - 일반 함수에서 Thread.sleep을 사용하여 블로킹 방식으로 지연을 구현합니다.
  - main 스레드가 3초 동안 완전히 멈추며, 이 시간 동안 다른 작업을 할 수 없습니다.
  - **의도:** suspend 함수와 비교하기 위한 기준점을 제공합니다. 블로킹 방식의 한계(UI 멈춤, 자원 낭비)를 보여줍니다.

---

## 결론 및 참고
- **suspend의 의의:**
  - 비동기/논블로킹 코드를 동기식처럼 작성 가능
  - 내부적으로 상태 머신으로 변환되어 효율적
- **참고 자료:**
  - [Kotlin 공식 문서: Suspend Functions](https://kotlinlang.org/docs/coroutines-basics.html#suspend-functions)
  - [Kotlin Coroutines: Under the hood](https://elizarov.medium.com/kotlin-coroutines-under-the-hood-4868d7abfc20) 