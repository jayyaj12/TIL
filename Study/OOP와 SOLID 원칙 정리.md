
# OOP와 SOLID: 특징별 개념 & 코틀린 예시 코드

> 안드로이드/서버 어디서든 통하는 기본기. 각 특징/원칙마다 **간단한 코드 예시**를 붙였어요.

---

## 1) OOP (객체 지향 프로그래밍) 4대 특징

### A. 추상화 (Abstraction)
핵심만 드러내고 구현 세부는 숨기기.

```kotlin
// 결제 수단의 핵심 능력만 노출
interface PaymentGateway {
    fun pay(amount: Long): Boolean
}

// 구현 세부(네트워크 호출/암호화 등)는 감춤
class CardGateway(private val token: String) : PaymentGateway {
    override fun pay(amount: Long): Boolean {
        // ... 토큰 검증, 카드사 API 호출 ...
        return true
    }
}

class KakaoPayGateway(private val userId: String) : PaymentGateway {
    override fun pay(amount: Long): Boolean {
        // ... 카카오페이 API 호출 ...
        return true
    }
}

// 클라이언트는 추상 타입만 알면 됨
class Checkout(private val gateway: PaymentGateway) {
    fun checkout(amount: Long) = gateway.pay(amount)
}
```

---

### B. 캡슐화 (Encapsulation)
데이터를 숨기고, 유효 상태만 유지되도록 보호.

```kotlin
class Account private constructor(
    private var _balance: Long
) {
    val balance: Long get() = _balance

    fun deposit(amount: Long) {
        require(amount > 0) { "입금액은 양수" }
        _balance += amount
    }

    fun withdraw(amount: Long) {
        require(amount > 0 && amount <= _balance) { "잔액 부족" }
        _balance -= amount
    }

    companion object {
        fun create(initial: Long) = Account(initial.coerceAtLeast(0))
    }
}

// 외부에서 _balance 직접 변경 불가 → 항상 유효함
```

---

### C. 상속 (Inheritance)
상위의 공통 기능을 재사용/확장.

```kotlin
open class BaseActivity : AppCompatActivity() {
    fun logScreen(name: String) = Log.d("Screen", name)
}

class HomeActivity : BaseActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        logScreen("Home")
    }
}
```

> 과도한 상속은 결합도를 높임. 공통은 **합성(Composition)**이 더 나을 때가 많음.

---

### D. 다형성 (Polymorphism)
같은 메시지 호출이 객체 타입에 따라 다르게 실행.

```kotlin
interface Shape { fun area(): Double }

class Circle(private val r: Double) : Shape {
    override fun area() = Math.PI * r * r
}
class Rect(private val w: Double, private val h: Double) : Shape {
    override fun area() = w * h
}

fun totalArea(shapes: List<Shape>) = shapes.sumOf { it.area() }
```

---

## 2) SOLID 5원칙

### S — SRP (단일 책임 원칙)
클래스는 **하나의 책임**만.

```kotlin
// ❌ 안 좋은 예: 저장 + 알림 + 검증을 한 클래스에 몰빵
class UserServiceBad {
    fun register(email: String) { /* 검증 + 저장 + 이메일전송 ... */ }
}

// ✅ 좋은 예: 책임 분리
class EmailValidator { fun isValid(email: String) = "@" in email }
interface UserRepository { fun save(email: String) }
interface Notifier { fun sendWelcome(email: String) }

class UserService(
    private val validator: EmailValidator,
    private val repo: UserRepository,
    private val notifier: Notifier
) {
    fun register(email: String) {
        require(validator.isValid(email))
        repo.save(email)
        notifier.sendWelcome(email)
    }
}
```

---

### O — OCP (개방-폐쇄 원칙)
**확장에는 열려**, **수정에는 닫힌** 구조.

```kotlin
// 전략 패턴으로 새로운 할인정책 추가 시 기존 코드 수정 최소화
interface DiscountPolicy { fun apply(price: Int): Int }

class NoDiscount : DiscountPolicy { override fun apply(price: Int) = price }
class RateDiscount(private val rate: Double) : DiscountPolicy {
    override fun apply(price: Int) = (price * (1 - rate)).toInt()
}

class OrderService(private val policy: DiscountPolicy) {
    fun finalPrice(price: Int) = policy.apply(price)
}

// 새로운 정책 추가 예) N원 고정 할인
class FlatDiscount(private val cut: Int) : DiscountPolicy {
    override fun apply(price: Int) = (price - cut).coerceAtLeast(0)
}
```

---

### L — LSP (리스코프 치환 원칙)
하위 타입은 상위 타입을 **대체 가능**해야 함.

```kotlin
open class Bird { open fun fly(): String = "flying" }
class Sparrow : Bird()

// ❌ 나쁜 예: 펭귄은 못 날지만 Bird를 상속해 fly를 예외로 처리 → 치환 위반
class PenguinBad : Bird() {
    override fun fly(): String = throw UnsupportedOperationException()
}

// ✅ 좋은 예: 타입 분리로 계약 보존
interface Animal
interface FlyingBird : Animal { fun fly(): String }
class RealSparrow : FlyingBird { override fun fly() = "flying" }
class Penguin : Animal // 날지 않는 새는 FlyingBird 아님
```

---

### I — ISP (인터페이스 분리 원칙)
클라이언트는 **사용하지 않는 메서드에 의존 X**.

```kotlin
// ❌ 나쁜 예: 다목적 인터페이스, 일부 클라이언트에 과도한 의존 강요
interface MultiFunctionMachine { fun print(); fun scan(); fun fax() }

// ✅ 좋은 예: 역할별로 작게 분리
interface Printer { fun print() }
interface Scanner { fun scan() }

class SimplePrinter : Printer { override fun print() { /*...*/ } }
class OfficeMachine : Printer, Scanner {
    override fun print() { /*...*/ }
    override fun scan() { /*...*/ }
}
```

---

### D — DIP (의존성 역전 원칙)
상위 모듈은 **구현**이 아니라 **추상**에 의존.

```kotlin
// 도메인 추상
interface AuthGateway { suspend fun login(id: String, pw: String): Boolean }

// 저수준 구현 (네트워크, DB 등)
class NetworkAuthGateway(/* retrofit */) : AuthGateway {
    override suspend fun login(id: String, pw: String) = true // API 호출
}

// 고수준 모듈은 추상에 의존
class AuthUseCase(private val gateway: AuthGateway) {
    suspend operator fun invoke(id: String, pw: String) = gateway.login(id, pw)
}

// DI로 연결 (Hilt 예시)
@Module
@InstallIn(SingletonComponent::class)
object AuthModule {
    @Provides fun provideAuthGateway(): AuthGateway = NetworkAuthGateway()
}
```

---

## 보너스) 안드로이드 뷰모델에 SOLID 녹이기

```kotlin
// SRP: ViewModel은 화면 상태/이벤트만; 비즈니스는 UseCase로
@HiltViewModel
class LoginViewModel @Inject constructor(
    private val authUseCase: AuthUseCase
) : ViewModel() {

    private val _state = MutableStateFlow(LoginState())
    val state: StateFlow<LoginState> = _state

    private val _events = MutableSharedFlow<String>()
    val events: SharedFlow<String> = _events

    fun login(id: String, pw: String) = viewModelScope.launch {
        _state.update { it.copy(loading = true) }
        val ok = runCatching { authUseCase(id, pw) }.getOrDefault(false)
        _state.update { it.copy(loading = false, success = ok) }
        _events.emit(if (ok) "환영합니다!" else "로그인 실패")
    }
}

data class LoginState(val loading: Boolean = false, val success: Boolean = false)
```

---

## 요약 표

| 구분 | 핵심 아이디어 | 한 줄 예시 |
|---|---|---|
| 추상화 | 핵심만 노출, 구현 숨김 | `interface PaymentGateway` |
| 캡슐화 | 상태 보호, 불변/검증 | `private var _balance` + 메서드로 변경 |
| 상속 | 공통 재사용/확장 | `open class BaseActivity` |
| 다형성 | 같은 호출, 다른 동작 | `Shape.area()` 다르게 구현 |
| SRP | 한 클래스=한 책임 | 검증/저장/알림 분리 |
| OCP | 수정 없이 확장 | 전략 추가로 기능 확장 |
| LSP | 하위가 상위를 대체 | 날 수 없는 새는 `FlyingBird` X |
| ISP | 작은 인터페이스 여러 개 | `Printer`/`Scanner` 분리 |
| DIP | 구현 대신 추상 의존 | `AuthUseCase(AuthGateway)` |

---

## 적용 팁
- **상태는 StateFlow, 이벤트는 SharedFlow**로 구분.
- 도메인은 **표준 라이브러리 외 의존 금지**.
- 새 요구가 왔을 때 **수정보다 확장**을 먼저 떠올리기(전략/데코레이터 등).
- “이 클래스의 변경 이유가 몇 개인가?”로 SRP 점검.

# 정리
SOLID 원칙은 객체 지향 설계의 5가지 핵심 원칙입니다.
- SRP(단일 책임 원칙): 클래스나 모듈은 변경 사유가 하나여야 하며, 하나의 책임만 가져야 합니다.
- OCP(개방-폐쇄 원칙): 코드는 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 합니다. (새 기능은 상속·구성으로 추가, 기존 코드는 변경 최소화)
- LSP(리스코프 치환 원칙): 하위 클래스는 상위 클래스와 호환 가능해야 하며, 상위 타입을 하위 타입으로 교체해도 동작이 유지돼야 합니다.
- ISP(인터페이스 분리 원칙): 사용하지 않는 기능이 강제로 포함되지 않도록, 클라이언트별로 꼭 필요한 인터페이스만 제공해야 합니다.
- DIP(의존 역전 원칙): 상위 모듈은 하위 구현이 아닌 추상(인터페이스)에 의존해야 하며, 세부 구현은 추상에 맞춰 교체 가능하게 설계합니다.