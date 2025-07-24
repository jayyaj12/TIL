# Kotlin IN ACTION 3장 함수 정의와 호출

### 가변 인자 함수 
가변 인자 함수는 호출할 때 원하는 개수만큼 여러 값을 인자로 넘기면 배열에 그 값들을 넣어주는 언어 기능인 `가변 길이 인자`를 사용한다. 
```kotlin
fun <T> listVariableOf(vararg values: T): List<T>  = listOf(*values)


val list = listOf(1, 2, 3, 4, 5, 6)
listVariableOf(*list)
```
> 배열을 넘길땐 `*` Spread 연산자를 이용해야 한다.  

### 중위 호출 
함수 호출을 연산자처럼 쓸 수 있게 해주는 문법이다.

일반적인 함수 호출
```kotlin
a.plus(b)
```

중위 호출 사용
```kotlin
a plus b
```

### 중위 함수 만드는 법 
```kotlin
class Warrior(val name: String) {
    infix fun attack(target: String) {
        println("$name attacks $target!")
    }
}

val w = Warrior("용사")
w attack "드래곤"
```
> `infix` 키워드를 함수 앞에 붙히면 된다. 

### 중위 함수 조건
1. 클래스의 멤버 함수 이거나 확장 함수여야 한다. 
2. 매개변수는 하나만 있어야 한다.
3. `infix` 키워드를 앞에 붙여야 한다. 

### 구조 분해 선언
```kotlin 
infix fun Any.to(other: Any) = Pair(this, other)

val (number, name) = 1 to "중현"
println(number) // 1
println(name) // 중현
```
> `to` 함수는 `Pair`의 인스턴스를 반환한다. `Pair`의 내용을 갖고 두 변수를 즉시 초기화 할 수 있다. 