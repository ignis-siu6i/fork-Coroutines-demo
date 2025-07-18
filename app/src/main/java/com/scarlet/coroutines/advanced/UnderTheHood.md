# UnderTheHood

이 파일은 코루틴의 내부 동작 원리를 설명합니다. suspend 함수가 어떻게 상태 머신으로 변환되고, 라벨(label)을 통한 중단점 관리, continuation 기반의 실행 흐름을 간단한 예제로 보여줍니다.

---

## main 함수
```kotlin
@DelicateCoroutinesApi
fun main() {
    GlobalScope.launch {
        val result = foo(1)
        println(result)
    }
    Thread.sleep(2_000)
}

suspend fun foo(arg: Int): Double {
    // label 0
    var local1 = 1.0
    var local2 = 2.0
    val res1 = bar(arg, local1)
    //label 1
    val res2 = bar(arg + 1, local2 + 1.0)
    //label 2
    return res1 + res2
}

suspend fun bar(m: Int, n: Double): Double {
    // label 0
    delay(1_000)
    // label 1
    return m + n
}
```
- **설명:**
  - suspend 함수들이 어떻게 내부적으로 상태 머신으로 변환되는지 보여줍니다.
  - 각 suspend 지점(delay, 다른 suspend 함수 호출)에서 라벨이 생성됩니다.
  - foo 함수는 3개의 라벨(0, 1, 2)을 가지며, bar 함수는 2개의 라벨(0, 1)을 가집니다.
  - **의도:** 코루틴 컴파일러가 suspend 함수를 어떻게 변환하는지 내부 구조를 실험적으로 보여줍니다.

---

## 코루틴 상태 머신의 동작 과정
1. **Label 0 (foo)**: 로컬 변수 초기화, 첫 번째 bar 호출
2. **Label 0 (bar)**: delay 호출로 인한 suspend
3. **Label 1 (bar)**: delay 완료 후 재개, 결과 반환
4. **Label 1 (foo)**: 첫 번째 bar 결과 받음, 두 번째 bar 호출
5. **Label 0 (bar)**: 다시 delay 호출로 인한 suspend
6. **Label 1 (bar)**: 두 번째 delay 완료 후 재개, 결과 반환
7. **Label 2 (foo)**: 두 번째 bar 결과 받음, 최종 결과 계산 및 반환

---

## 결론 및 참고
- **코루틴 내부 원리:**
  - suspend 함수는 상태 머신으로 변환됨
  - 각 suspend 지점에서 라벨 생성
  - Continuation을 통한 상태 보존과 재개
  - 스택 없는 비동기 실행
- **실용적 의미:**
  - 메모리 효율적인 비동기 처리
  - 스레드 블로킹 없는 대기
  - 순차적 코드 스타일로 비동기 작성 가능
- **참고 자료:**
  - [Kotlin Coroutines: Under the hood](https://elizarov.medium.com/kotlin-coroutines-under-the-hood-4868d7abfc20)
  - [Deep Dive into Coroutines on JVM](https://www.youtube.com/watch?v=YrrUCSi72E8) 