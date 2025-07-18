# B04_Launch

## 1. launch를 이용한 코루틴 생성

```kotlin
runBlocking {
    launch {
        log("3. before save")
        save(User("A001", "Jody", 33))
        log("4. after save")
    }
    log("2. after launch")
}
```
- **설명:**
  launch는 새로운 코루틴을 생성해 비동기 작업을 실행합니다. runBlocking 내에서 launch를 사용하면, 메인 스레드가 코루틴의 완료를 기다립니다.

---

## 2. 여러 코루틴의 동시 실행과 join

```kotlin
runBlocking {
    val job1 = launch { delay(1_000); log("child 1 done.") }
    val job2 = launch { delay(2_000); log("child 2 done.") }
    job1.join(); job2.join()
    log("모든 작업 완료")
}
```
- **설명:**
  여러 코루틴을 동시에 실행하고, join을 통해 특정 코루틴의 완료를 기다릴 수 있습니다.

---

## 3. GlobalScope와 CoroutineScope의 차이

```kotlin
GlobalScope.launch {
    // 앱 전체 생명주기와 무관하게 동작 (권장하지 않음)
}

val scope = CoroutineScope(Job())
val job = scope.launch {
    // scope가 살아있는 동안만 동작
}
```
- **설명:**
  GlobalScope는 앱 전체와 생명주기를 같이 하므로, 메모리 누수나 예측 불가한 동작이 발생할 수 있습니다. CoroutineScope를 명시적으로 관리하는 것이 안전합니다.

---

## 4. 결론 및 참고

- **launch의 용도:**
  - 비동기 작업, 구조적 동시성 구현에 필수
  - scope와 함께 사용해 생명주기 관리

- **참고 자료:**
  - [Kotlin 공식 문서: launch](https://kotlinlang.org/docs/coroutines-basics.html#launching-a-coroutine) 