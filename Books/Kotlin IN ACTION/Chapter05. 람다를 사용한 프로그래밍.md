### 함수형 프로그래밍 특징
1. __일급 시민인 함수__: 함수를 값으로 다룰 수 있다. 함수를 변수에 저장하고 파라미터로 전달하며 함수에서 다른 함수를 반환할 수 있다. 
2. __불변성__: 객체를 만들 때 일단 만들어진 다음에는 내부 상태가 변하지 않음을 보장한다.
3. __부수 효과 없음__: 함수가 똑같은 입력에 대해 항상 같은 출력을 내놓고 다른 객체나 외부 세계의 상태를 변경하지 않게 구성한다. 

```kotlin
// with는 객체를 인자로 받아 블록 안에서 작업한 후, 마지막 표현식의 값을 반환하는 함수이다.
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }

    append("\nNow I know the alphabet!")
    toString()
}

// apply는 객체를 설정(configure)할 때 사용하며, 블록 실행 후 해당 객체 자신을 그대로 반환하는 함수이다.
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }

    append("\nNow I know the alphabet!")
}
```
