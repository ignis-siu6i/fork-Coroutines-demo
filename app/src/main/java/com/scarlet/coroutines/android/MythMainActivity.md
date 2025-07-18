# MythMainActivity

## 1. lifecycleScope와 UI 비동기 작업

```kotlin
findButton.setOnClickListener {
    primeJob = lifecycleScope.launch {
        val primeNumber = findBigPrime_Wish_To_Be_NonBlocking()
        status.text = primeNumber.toString()
    }
}
```
- **설명:**
  Android에서 lifecycleScope를 사용하면 Activity/Fragment의 생명주기에 맞춰 코루틴을 안전하게 실행할 수 있습니다. UI 업데이트, 네트워크 요청 등 비동기 작업을 안전하게 처리합니다.

---

## 2. repeatOnLifecycle로 안전한 반복 작업

```kotlin
countingJob = lifecycleScope.launch {
    lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
        var value = 0
        while (true) {
            textView.text = value.toString()
            value++
            delay(1_000)
        }
    }
}
```
- **설명:**
  repeatOnLifecycle을 사용하면 특정 생명주기 상태에서만 반복 작업을 실행할 수 있습니다. 화면이 비활성화되면 자동으로 일시 중단되어 메모리 누수와 불필요한 작업을 방지합니다.

---

## 3. 취소와 UI 반영

```kotlin
cancelButton.setOnClickListener {
    primeJob?.cancel()
    status.text = "findBigPrime cancelled"
}
```
- **설명:**
  코루틴을 명시적으로 취소하고, UI에 즉시 반영할 수 있습니다. 코루틴의 취소는 안전하게 처리되며, UI와의 연동이 자연스럽습니다.

---

## 4. 결론 및 참고

- **Android에서 코루틴의 활용:**
  - 생명주기 안전, UI/비동기 작업, 반복 작업, 취소 등

- **참고 자료:**
  - [공식 문서: lifecycleScope](https://developer.android.com/topic/libraries/architecture/coroutines#lifecyclescope)
  - [공식 문서: repeatOnLifecycle](https://developer.android.com/topic/libraries/architecture/coroutines#repeatonlifecycle) 