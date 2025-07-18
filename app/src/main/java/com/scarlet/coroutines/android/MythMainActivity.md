# MythMainActivity

이 파일은 Android 환경에서 코루틴의 잘못된 사용법과 올바른 사용법을 비교합니다. UI 스레드에서 CPU 집약적 작업을 수행할 때의 문제점과 withContext(Dispatchers.Default)를 사용한 해결책, lifecycleScope와 repeatOnLifecycle의 사용법을 보여줍니다.

---

## onCreate - UI 스레드 블로킹 문제 시연
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    findButton.setOnClickListener {
        primeJob = lifecycleScope.launch {
            val primeNumber = findBigPrime_Wish_To_Be_NonBlocking()
            // val primeNumber = findBigPrime_ProperWay()
            status.text = primeNumber.toString()
        }
    }
    
    countingJob = lifecycleScope.launch {
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            var value = 0
            while (true) {
                textView.text = value.toString().also { value++ }
                delay(1_000)
            }
        }
    }
}
```
- **설명:**
  - findButton 클릭 시 lifecycleScope에서 큰 소수를 찾는 CPU 집약적 작업을 수행합니다.
  - 동시에 카운터가 1초마다 증가하는 UI 업데이트 작업도 실행됩니다.
  - **의도:** UI 스레드에서 CPU 집약적 작업을 할 때의 문제점과 올바른 해결책을 비교 실험으로 보여줍니다.

---

## findBigPrime_Wish_To_Be_NonBlocking vs findBigPrime_ProperWay
```kotlin
// 잘못된 방법 - UI 스레드를 블로킹함
private suspend fun findBigPrime_Wish_To_Be_NonBlocking(): BigInteger =
    BigInteger.probablePrime(4096, Random())

// 올바른 방법 - 백그라운드 스레드에서 실행
private suspend fun findBigPrime_ProperWay(): BigInteger = withContext(Dispatchers.Default) {
    BigInteger.probablePrime(4096, Random())
}
```
- **설명:**
  - **잘못된 방법**: suspend 함수라고 해서 자동으로 백그라운드에서 실행되지 않습니다. BigInteger.probablePrime은 블로킹 함수이므로 UI 스레드를 멈춥니다.
  - **올바른 방법**: withContext(Dispatchers.Default)로 CPU 집약적 작업을 백그라운드 스레드로 전환합니다.
  - **의도:** suspend 함수의 일반적인 오해를 바로잡고 적절한 디스패처 사용의 중요성을 실험적으로 보여줍니다.

---

## lifecycleScope와 repeatOnLifecycle 사용
```kotlin
countingJob = lifecycleScope.launch {
    lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
        var value = 0
        while (true) {
            textView.text = value.toString().also { value++ }
            delay(1_000)
        }
    }
}
```
- **설명:**
  - lifecycleScope는 Activity/Fragment의 생명주기와 연결된 코루틴 스코프입니다.
  - repeatOnLifecycle(Lifecycle.State.STARTED)는 STARTED 상태에서만 실행되고, STOPPED 상태가 되면 일시 중단됩니다.
  - **의도:** Android에서 생명주기 인식 코루틴 사용법과 메모리 누수 방지를 실험적으로 보여줍니다.

---

## 취소 처리와 리소스 정리
```kotlin
cancelButton.setOnClickListener {
    primeJob?.cancel()
}

override fun onDestroy() {
    super.onDestroy()
    countingJob?.cancel()
    exitProcess(0)
}
```
- **설명:**
  - 사용자가 취소 버튼을 누르면 진행 중인 소수 찾기 작업을 취소합니다.
  - onDestroy에서 카운팅 Job을 취소하여 메모리 누수를 방지합니다.
  - **의도:** Android에서 코루틴의 적절한 취소와 리소스 정리 패턴을 실험적으로 보여줍니다.

---

## 실험 시나리오
1. **잘못된 방법 테스트**: `findBigPrime_Wish_To_Be_NonBlocking()` 사용 시 UI가 멈추는지 확인
2. **올바른 방법 테스트**: `findBigPrime_ProperWay()` 사용 시 UI가 계속 반응하는지 확인
3. **생명주기 테스트**: 앱을 백그라운드로 보냈다가 다시 포그라운드로 가져올 때 카운터 동작 확인

---

## 결론 및 참고
- **Android 코루틴 사용 원칙:**
  - UI 스레드에서 블로킹 작업 금지
  - 적절한 디스패처 사용 (Default, IO, Main)
  - lifecycleScope와 repeatOnLifecycle 활용
  - 적절한 취소와 리소스 정리
- **참고 자료:**
  - [Android 공식 문서: Coroutines on Android](https://developer.android.com/kotlin/coroutines)
  - [Lifecycle-aware components](https://developer.android.com/topic/libraries/architecture/lifecycle) 