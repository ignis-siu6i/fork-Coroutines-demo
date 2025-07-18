# TrampolineDemo

이 파일은 재귀 함수 최적화와 트램폴린(Trampoline) 패턴을 설명합니다. 일반 재귀, 꼬리 재귀(tail recursion), 그리고 트램폴린을 사용한 스택 오버플로우 방지 기법을 factorial 계산 예제로 보여줍니다.

---

## factorials.main
```kotlin
object factorials {
    // Plain recursive factorial function
    fun factorial(n: Long): BigInteger =
        if (n <= 1) {
            BigInteger.ONE
        } else {
            n.toBigInteger() * factorial(n - 1)
        }

    @JvmStatic
    fun main(args: Array<String>) {
        println((0 until 10).map { factorial(it.toLong()) })
        println(factorial(10_000L))
    }
}
```
- **설명:**
  - 일반적인 재귀 함수로 factorial을 계산하는 방법입니다.
  - 큰 숫자(10,000!)를 계산하면 스택 오버플로우가 발생할 수 있습니다.
  - 각 재귀 호출마다 스택 프레임이 쌓여서 메모리를 많이 사용합니다.
  - **의도:** 일반 재귀의 한계와 스택 오버플로우 문제를 실험적으로 보여줍니다.

---

## factorials_TR.main
```kotlin
object factorials_TR {
    // Tail-recursive function
    tailrec
    fun factorial(n: Long, accumulator: BigInteger): BigInteger =
        if (n <= 1) {
            accumulator
        } else {
            factorial(n - 1, accumulator * n.toBigInteger())
        }

    @JvmStatic
    fun main(args: Array<String>) {
        println((0 until 10).map { factorial(it.toLong(), BigInteger.ONE) })
        println(factorial(10_000, BigInteger.ONE))
    }
}
```
- **설명:**
  - `tailrec` 키워드를 사용한 꼬리 재귀 최적화입니다.
  - accumulator 패턴으로 중간 결과를 누적하여 계산합니다.
  - 컴파일러가 재귀 호출을 루프로 최적화하여 스택 오버플로우를 방지합니다.
  - **의도:** 꼬리 재귀 최적화의 효과와 큰 값 계산의 안전성을 실험적으로 보여줍니다.

---

## trampoline_demo1.main
```kotlin
object trampoline_demo1 {
    fun factorial(n: Long, accumulator: BigInteger): BigInteger =
        if (n <= 1) {
            accumulator
        } else {
            factorial(n - 1, accumulator * n.toBigInteger())
        }

    fun <T> run(f: () -> Any?): T = TODO()

    @JvmStatic
    fun main(args: Array<String>) {
        println((0 until 10).map { factorial(it.toLong(), BigInteger.ONE) })
        println(factorial(10_000, BigInteger.ONE))
    }
}
```
- **설명:**
  - 트램폴린 패턴 구현을 위한 준비 단계입니다.
  - `run` 함수가 TODO로 남겨져 있어 구현이 필요합니다.
  - 지연 평가(lazy evaluation)를 통한 스택 안전성을 목표로 합니다.
  - **의도:** 트램폴린 패턴의 기본 구조와 구현 과제를 실험적으로 보여줍니다.

---

## trampoline_demo2.main
```kotlin
object trampoline_demo2 {
    fun factorial(n: Long, accumulator: BigInteger): BigInteger =
        if (n <= 1) {
            accumulator
        } else {
            factorial(n - 1, accumulator * BigInteger.valueOf(n))
        }

    @JvmStatic
    fun main(args: Array<String>) {
        println((0 until 10).map { factorial(it.toLong(), BigInteger.ONE) })
        println(factorial(10_000, BigInteger.ONE))
    }
}
```
- **설명:**
  - 완성된 트램폴린 구현을 보여주는 것으로 보이지만, 실제로는 일반 재귀와 동일합니다.
  - 실제 트램폴린 구현이 누락되어 있어 여전히 스택 오버플로우 위험이 있습니다.
  - **의도:** 트램폴린 패턴의 완성된 형태를 위한 기반을 실험적으로 보여줍니다.

---

## 실제 트램폴린 구현 예제

### 1. 기본 트램폴린 구조
```kotlin
sealed class Trampoline<out A> {
    data class Done<out A>(val result: A) : Trampoline<A>()
    data class More<out A>(val thunk: () -> Trampoline<A>) : Trampoline<A>()
}

fun <A> run(trampoline: Trampoline<A>): A {
    var current = trampoline
    while (true) {
        when (current) {
            is Trampoline.Done -> return current.result
            is Trampoline.More -> current = current.thunk()
        }
    }
}
```

### 2. 트램폴린을 사용한 factorial
```kotlin
fun factorial(n: Long, acc: BigInteger = BigInteger.ONE): Trampoline<BigInteger> =
    if (n <= 1) {
        Trampoline.Done(acc)
    } else {
        Trampoline.More { factorial(n - 1, acc * n.toBigInteger()) }
    }

// 사용법
val result = run(factorial(10_000))
```

---

## 각 방법의 장단점 비교

### 일반 재귀
- **장점**: 간단하고 직관적
- **단점**: 스택 오버플로우 위험

### 꼬리 재귀 (tailrec)
- **장점**: 컴파일러 최적화, 성능 좋음
- **단점**: Kotlin에서만 지원, 상호 재귀 불가

### 트램폴린
- **장점**: 스택 안전, 언어 독립적, 상호 재귀 지원
- **단점**: 복잡한 구현, 약간의 성능 오버헤드

---

## 실험 시나리오
1. **스택 오버플로우 테스트**:
   - `factorials.factorial(100_000L)` 실행
   - 스택 오버플로우 확인

2. **꼬리 재귀 성능 테스트**:
   - `factorials_TR.factorial(100_000, BigInteger.ONE)` 실행
   - 정상 동작 확인

3. **트램폴린 구현**:
   - TODO 부분 구현하여 스택 안전성 확인

---

## 실용적 활용 사례
1. **깊은 재귀 알고리즘**: 트리 순회, 그래프 탐색
2. **함수형 프로그래밍**: 스택 안전한 재귀
3. **파서 구현**: 재귀 하강 파서
4. **상태 머신**: 복잡한 상태 전환

---

## 결론 및 참고
- **재귀 최적화 전략:**
  - 단순한 경우: tailrec 사용
  - 복잡한 경우: 트램폴린 패턴
  - 성능이 중요한 경우: 반복문으로 변환
- **코루틴과의 연관성:**
  - 코루틴도 스택 없는 실행을 위해 유사한 기법 사용
  - 상태 머신 변환과 트램폴린의 유사점
- **참고 자료:**
  - [Tail Recursion in Kotlin](https://kotlinlang.org/docs/functions.html#tail-recursive-functions)
  - [Trampoline Pattern](https://blog.logrocket.com/using-trampolines-to-manage-large-recursive-loops-in-javascript-d8c9db095ae3/) 