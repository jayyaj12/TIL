## data class 란 ? 
코틀린에서 데이터만 담기 위한 클래스를 간단하게 정의할 수 있도록 제공되는 클래스  
```kotlin 
data class User(val name: String, val age: Int)
``` 

### data class가 자동으로 해주는 것 
|기능|설명|
|---|---|
|`equals()` |값 기준으로 객체 비교 가능|
|`hashCode()` |HashMap, Set 등에서 정상 동작|
|`toString()` |보기 좋은 문자열 출력|
|`copy()` | 일부만 바꿔서 새로운 객체 만들기|
|`componentN()` |구조 분해 지원(ex: val (n, a) = user)|

## 일반 클래스와 data class 비교
### 일반 클래스
```kotlin
class User(val name: String, val age: Int)

val u1 = User("철수", 20)
val u2 = User("철수", 20)

println(u1 == u2) // false (동일성 비교)
```

### data class
```kotlin
data class User(val name: String, val age: Int)

val u1 = User("철수", 20)
val u2 = User("철수", 20)

println(u1 == u2) // true (내용 같으니 동등성 비교)
``` 
> 일반 클래스는 `equals()` 함수를 override 하지 않았으므로 객체의 참조 주소만을 비교하여 `false`가 나오고,  
data class 는 자동으로 `equals()` 함수를 override 하였으므로 객체의 값이 동일하다면 `true`가 나온다. 