## GC(Garbage Collection) 란 ?
자바에서 더 이상 사용되지 않는 객체를 자동으로 찾아서 메모리에서 제거 하는 기능

## GC(Garbage Collection) 역할
- 자바는 `new` 키워드로 객체를 생성하면 힙 메모리에 저장한다.
- 객체를 더 이상 참조하지 않으면 쓸데 없는 메모리 낭비를 하게 된다.
- 이걸 자동으로 정리해주는게 GC(Garbage Collection)이다. 

## GC(Garbage Collection)가 동작하는 순간 ? 
- 힙 공간이 부족할 때
- 명시작으로 `System.gc()` 를 호출할 때 (강제 아님)
- JVM이 적절하다고 판단할 때 

## 자동 정리 우선 순위를 높이는 방법 
```java
String s = new String("Hello");
s = null // "Hello" 는 더 이상 접근할 방법이 없다. -> GC 대상
``` 
일반적으로 null 로 만들면 우선 순위가 높아진다. <br>
위 처럼 더 이상 변수나 객체에서도 참조되지 않는 객체를 자동 정리한다. 

## JVM이 GC가 적절하다고 판단할 때 ?
자바에서는 GC 실행 시점이 정해져 있는 것이 아니고, JVM 내부 알고리즘이 다양한 신호를 보고 결정한다.
<br>[조건]
1. **힙 메모리 부족**: 객체를 할당하려고 보니 Young 또는 Old 영역이 부족할 때
2. **Eden 영역이 꽉 찼을 때**: Young Generation의 Eden 공간이 꽉 차면 Minor GC가 발생한다.
3. **Old 영역이 일정 임계치 초과**: Full GC 또는 Major GC 트리거
4. **메모리 할당 실패**: `outOfMemoryError` 전에 마지막 시도로 GC 를 실행
5. **JVM 내부 힌트**: GC의 지연 시간, 힙 조각화 상태, 참조 수 감소 등 내부 메트릭을 분석해 실행한다. 

## System.gc() 가 "강제가 아닌 이유" ?
`System.gc()` 메서드를 실행하면 JVM 으로 GC 요청을 보내서 JVM이 판단했을 때 필요치 않다면 실행하지 않는다.<br>
그 이유로는 아래와 같다.
1. GC는 비용이 큰 작업이라 성능 저하 위험이 있다.
2. JVM 옵션에 따라 무시가 가능하다. 

## 힙 메모리
자바에서 `new` 키워드로 객체를 만들면 Heap 메모리에 저장된다. GC가 관리하는 대상은 바로 힙 영역이다.<bR>
힙 영역은 크게 Young Generation 과 Old Generation 으로 나뉘어 있다.

## Eden 영역(Eden Scope)
Young Generation 의 한 부분으로 Young Generation 은 새로 생성된 객체들이 먼저 저장되는 공간이고, 그중에서 가장 앞단에 있는 공단이 바로 Eden 이다. <br>
대부분의 객체는 Eden 영역에 잠깐 있다가 사라진다. Eden 이 꽉차면 Minor GC 가 발생하고, 살아남은 객체는 Survivor 영역으로 이동한다.

## Old 영역(Tenured Generation)
Young Generation 에서 여러 번 거쳐 살아남은 객체들이 가는 곳이다.<br>
Old 영역은 크고, 변화가 작다. Old 영역이 꽉차면 Major GC 또는 Full GC가 발생한다. 

