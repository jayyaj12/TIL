##  Kotlin 은 pass-by-value 언어이다. 
kotlin은 모든 함수 인자 전달 방식은 pass-by-value(값에 의한 전달)이다.  

## 1. 기본 타입(Int, Double 등) -> 진짜 값 자체를 복사해서 전달한다.
```kotlin
fun changeValue(x: Int) {
    var x = x
    x += 1
    println("함수 안: $x")
}

fun main() {
    val a = 10
    changeValue(a)
    println("함수 밖: $a")  // 여전히 10
}
```
> `Int`는 기본형(primitive)이니까, 그냥 a 값 즉 10을 복사해서 함수로 전달한다.  
값을 전달했기 때문에 원본 값은 변경되지 않는다.

## 2.객체(class 등) -> 참조(reference)를 값으로 전달
```kotlin
data class Person(var name: String)

fun changeName(p: Person) {
    p.name = "변경됨"
}

fun main() {
    val person = Person("원래 이름")
    changeName(person)
    println(person.name) // 변경됨
}
``` 
> 여기서도 pass-by-value 이지만 여기서 복사된 것은 값이 아닌 **참조(주소)** 인 것이다. 그래서 함수 안에서 객체 내부 속성 `name`을 바꾸면 진짜 객체가 변경되는 것처럼 보인다. 

`p` 변수를 통해 내부 속성은 변경할 수 있지만 참조 자체를 바꿔도 원본 변수엔 영향이 없다. 아래에서 살펴보자 
```kotlin 
data class Person(var name: String)

fun changeName(p: Person) {
    p = Person("개명")
}

fun main() {
    val person = Person("원래 이름")
    changeName(person)
    println(person.name) // 원래 이름 변경 안됨 
}
``` 

하지만 `changeName()` 함수 안에서 `p`는 person과 같은 객체를 가르키지만, 참조값 자체는 복사된 것이기 때문에 `p = Person("개명)`을 해도 `person` 에는 영향이 없다.