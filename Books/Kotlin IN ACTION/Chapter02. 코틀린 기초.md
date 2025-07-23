# Kotlin IN ACTION 2장 코틀린 기초 정리 

### 블록 본문 함수와 식 본문 함수
블록 본문 함수는 본문이 중괄호로 둘러싸인 함수를 말한다.
```kotlin
fun max(a: Int, b:Int): Int {
    return if(a > b) a else b
}
```  

식 본문 함수는 등호와 식으로 이뤄진 함수이다.
```kotlin
fun max(a: Int, b: Int): Int = if(a > b) a else b 
``` 
코틀린에서는 식 본문 함수가 자주 쓰인다.