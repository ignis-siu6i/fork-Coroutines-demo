# B02_CPS

이 파일은 Continuation Passing Style(CPS, 연속 전달 스타일) 개념을 소개합니다. 함수형 프로그래밍에서의 CPS 변환, 재귀 함수(팩토리얼, 피보나치 등)를 CPS로 변환하는 연습을 통해 비동기 흐름 제어의 기초를 설명합니다.

---

## Continuation_Passing_Style.main
```kotlin
object Continuation_Passing_Style {
    @JvmStatic
    fun main(args: Array<String>) {
        println(evaluate())
        println(fact(10))
        println((0..10).map { fib(it.toLong()) }.joinToString(", "))
    }
}
```
- **설명:**
  - `evaluate()`는 단순한 수식 계산을 순차적으로 수행합니다. (1 + 2) * (3 + 4)와 같은 계산을 Label별로 나눠서 보여줍니다.
  - `fact(10)`은 재귀적으로 팩토리얼을 계산합니다. `fib(n)`은 피보나치 수열을 재귀적으로 계산합니다.
  - **의도:**
    - 일반적인 재귀 함수와 CPS 스타일의 차이를 이해하고, CPS 변환 연습을 통해 비동기/콜백 기반 프로그래밍의 기초를 익히는 데 목적이 있습니다.
    - 실제로 CPS 스타일로 변환하면 콜백 지옥, 비동기 흐름 제어, 코루틴 내부 동작 등과 연결됩니다.

---

## 결론 및 참고
- **CPS의 의의:**
  - 비동기/콜백 기반 프로그래밍의 기초가 되는 패턴
  - 함수형 프로그래밍, 코루틴 내부 구현 등에서 활용
- **참고 자료:**
  - [Wikipedia: Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)
  - [Kotlin Coroutines: Continuations](https://kotlinlang.org/docs/coroutines-intrinsics.html) 