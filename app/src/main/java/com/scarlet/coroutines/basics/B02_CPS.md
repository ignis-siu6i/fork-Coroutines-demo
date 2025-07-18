# B02_CPS

## 1. 일반 계산과 CPS(Continuation Passing Style) 비교

```kotlin
private fun evaluate(): Double {
    val step1 = add(1, 2)
    val step2 = add(3, 4)
    val step3 = mul(step1.toDouble(), step2.toDouble())
    return step3
}
```
- **설명:**
  일반적인 함수 호출 방식으로, 계산이 순차적으로 진행됩니다. CPS로 변환하면 각 단계의 결과를 다음 함수에 넘기며, 비동기 흐름 제어나 콜백 스타일과 유사한 구조를 만들 수 있습니다.

---

## 2. 재귀 함수의 CPS 변환 연습

```kotlin
private fun fact(n: Long): Long =
    when (n) {
        0L -> 1L
        else -> n * fact(n - 1)
    }

private fun fib(n: Long): Long =
    when (n) {
        0L, 1L -> n
        else -> fib(n - 1) + fib(n - 2)
    }
```
- **설명:**
  팩토리얼과 피보나치 수열을 재귀적으로 구현한 예시입니다. 이 코드를 CPS 스타일로 변환하면, 비동기 콜백이나 코루틴의 흐름 제어와 유사한 패턴을 연습할 수 있습니다.

---

## 3. 결론 및 참고

- **CPS의 의의:**
  - 비동기/콜백 기반 프로그래밍의 기초가 되는 패턴
  - 함수형 프로그래밍, 코루틴 내부 구현 등에서 활용

- **참고 자료:**
  - [Wikipedia: Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)
  - [Kotlin Coroutines: Continuations](https://kotlinlang.org/docs/coroutines-intrinsics.html) 