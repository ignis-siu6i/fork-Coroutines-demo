# B05_Async

## 1. async/await를 이용한 비동기 결과 수집

```kotlin
val deferred = async {
    log("\tRequest user with Id A001")
    getUser("A001")
}
val user = deferred.await()
log("Returned user = $user")
```
- **설명:**
  async는 결과를 반환하는 비동기 작업에 사용합니다. await를 호출하면 결과가 준비될 때까지 일시 중단(suspend)됩니다.

---

## 2. GlobalScope.async의 위험성

```kotlin
GlobalScope.async {
    // 앱 전체 생명주기와 무관하게 동작 (권장하지 않음)
}
```
- **설명:**
  GlobalScope에서 async를 사용하면, 결과를 안전하게 수집하지 못하거나 메모리 누수 위험이 있습니다. 항상 scope를 명확히 관리하세요.

---

## 3. 결론 및 참고

- **async/await의 용도:**
  - 비동기적으로 값을 반환받고 싶을 때 사용
  - launch와 달리 결과(Deferred)를 다룸

- **참고 자료:**
  - [Kotlin 공식 문서: async](https://kotlinlang.org/docs/coroutines-basics.html#async-coroutine-builder) 