# Kotlin 비동기 스트림 완전 정리: Flow vs StateFlow vs SharedFlow vs LiveData

## 1. LiveData
**개념**  
Android **라이프사이클 인식** 데이터 홀더. Activity/Fragment가 활성 상태일 때만 데이터를 전달합니다.

**특성**
- **핫 스트림**: 항상 최신 값을 보유
- **라이프사이클 자동 관리**: 화면이 백그라운드면 콜백 중지
- **스레드 제약**: `setValue`는 메인 스레드, `postValue`는 백그라운드 가능

**장점**
- View 계층에서 바로 사용 가능 (`observe`만 호출하면 됨)
- 라이프사이클 처리 내장 → 메모리 누수 위험 적음

**단점**
- 코루틴/Flow 연산자 활용 어려움
- 비-Android 환경에선 사용 불가

**주요 용도**
- 단순 UI 데이터 바인딩
- 기존 MVVM 코드베이스 유지보수

**코드 예시**
```kotlin
private val _count = MutableLiveData(0)
val count: LiveData<Int> = _count

fun increment() {
    _count.value = (_count.value ?: 0) + 1
}

// View
viewModel.count.observe(viewLifecycleOwner) { value ->
    textView.text = value.toString()
}
```

---

## 2. Flow
**개념**  
Kotlin Coroutines에서 제공하는 **콜드 스트림**. `collect`할 때마다 데이터 소스를 새로 실행.

**특성**
- **콜드**: 구독할 때마다 새로 시작
- **초기값 없음**
- **백프레셔 자연 처리** (suspend 기반)
- 예외는 스트림 안에서 처리 가능 (`catch`)

**장점**
- 네트워크/DB 쿼리처럼 매번 새로 실행이 필요한 경우에 적합
- 풍부한 연산자(map, filter, debounce 등)

**단점**
- 구독자 수만큼 데이터 소스 재실행
- 라이프사이클 인식 없음 → 별도 처리 필요

**주요 용도**
- 일회성 계산, 네트워크 호출, DB 스트림

**코드 예시**
```kotlin
fun fetchNumbers(): Flow<Int> = flow {
    for (i in 1..3) {
        emit(i)
        delay(1000)
    }
}

// View
lifecycleScope.launch {
    fetchNumbers()
        .onStart { showLoading() }
        .catch { showError(it) }
        .collect { updateUI(it) }
}
```

---

## 3. StateFlow
**개념**  
항상 **최신 상태**를 보관하는 **핫 스트림**. `LiveData`의 코루틴 버전.

**특성**
- **핫**: 항상 동작, 멀티 구독 가능
- **초기값 필수**
- 구독 시 최신 상태 즉시 전달

**장점**
- 화면 상태 모델링에 최적
- 재구독 시 바로 최신값 전달

**단점**
- 1회성 이벤트엔 부적합 (재구독 시 재전달)

**주요 용도**
- UI 상태(로딩, 에러, 데이터)

**코드 예시**
```kotlin
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState

fun updateName(name: String) {
    _uiState.update { it.copy(name = name) }
}

// View
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.uiState.collect { state ->
            nameTextView.text = state.name
        }
    }
}
```

---

## 4. SharedFlow
**개념**  
다중 구독 가능한 **핫 브로드캐스트 스트림**. `replay`/버퍼 정책 설정 가능.

**특성**
- 초기값 불필요
- `replay=0` 설정 시 1회성 이벤트 처리 가능

**장점**
- 토스트, 네비게이션 이벤트 같은 일회성 동작에 적합
- 다중 구독 지원

**단점**
- 잘못된 `replay` 설정 시 스티키 이벤트 발생 가능

**주요 용도**
- UI 이벤트, 브로드캐스트

**코드 예시**
```kotlin
private val _events = MutableSharedFlow<String>(replay = 0)
val events: SharedFlow<String> = _events

fun sendToast(message: String) {
    viewModelScope.launch { _events.emit(message) }
}

// View
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.events.collect { msg ->
            Toast.makeText(context, msg, Toast.LENGTH_SHORT).show()
        }
    }
}
```

---

## 비교 표

| 특징            | Flow (콜드) | StateFlow (핫) | SharedFlow (핫) | LiveData (핫) |
|----------------|------------|---------------|----------------|--------------|
| 초기값          | ❌         | ✅            | ❌             | ✅           |
| 재생값          | ❌         | 최신 1개      | `replay` 설정   | 현재 값      |
| 라이프사이클 인식 | ❌         | ❌            | ❌             | ✅           |
| 멀티 구독       | 소스 재실행 | 공유           | 공유            | 공유         |
| 주 사용처       | 쿼리, 파이프라인 | UI 상태        | 이벤트         | UI 바인딩    |

---

## 선택 가이드
- **UI 상태**: `StateFlow`
- **UI 이벤트(1회성)**: `SharedFlow`(`replay=0`)
- **데이터 쿼리/계산**: `Flow`
- **기존 View 연동**: `LiveData` 또는 `asLiveData()`
