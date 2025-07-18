# A05_Dispatchers

## 1. Dispatcher 종류와 특징

```kotlin
launch(Dispatchers.Default) { /* CPU-bound 작업 */ }
launch(Dispatchers.IO) { /* IO-bound 작업 */ }
launch(Dispatchers.Main) { /* UI 스레드 */ }
launch(Dispatchers.Unconfined) { /* 특별한 정책 없음 */ }
```
- **설명:**
  - Default: CPU 연산에 적합, 코어 수만큼 스레드 풀
  - IO: 네트워크/디스크 IO에 적합, 많은 스레드 풀
  - Main: UI 스레드(Android 등)
  - Unconfined: 특정 스레드에 묶이지 않음(권장X)

---

## 2. Dispatcher 커스텀 및 병렬성 제한

```kotlin
val context = newSingleThreadContext("CustomDispatcher")
launch(context) { /* ... */ }
val limitedDispatcher = Dispatchers.IO.limitedParallelism(4)
launch(limitedDispatcher) { /* ... */ }
```
- **설명:**
  커스텀 스레드 풀이나 병렬성 제한을 통해, 자원 사용을 세밀하게 제어할 수 있습니다.

---

## 3. Dispatcher 간 스레드 전환

```kotlin
withContext(Dispatchers.IO) { /* IO 작업 */ }
withContext(Dispatchers.Default) { /* CPU 작업 */ }
```
- **설명:**
  withContext를 사용하면 코루틴 내에서 Dispatcher를 전환할 수 있습니다.

---

## 4. 결론 및 참고

- **Dispatcher의 활용:**
  - 작업 특성에 맞는 Dispatcher 선택, 커스텀 스레드 풀, 병렬성 제한 등

- **참고 자료:**
  - [Kotlin 공식 문서: Dispatchers](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html#dispatchers-and-threads) 