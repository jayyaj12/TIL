## Dynamic Array는 어떤 자료구조인가요?
Dynamic Array는 `size를 자동적으로 resizing 하는 Array 입니다`. 기존에 고정된 size를 가진 Static Array의 한계점을 보안하고자 고안되었습니다. Dynamic Array는 data를 계속 추가하다가 기존에 할당된 memory 를 초과하게 되면, `size를 늘린 배열을 선언하고 그곳으로 모든 데이터를 옮김으로써 늘어난 크기의 size 를 가진 배열이 됩니다. 이를 resize 라고 합니다.`
이로써 새로운 data 를 저장할 수 있게 됩니다. 따라서 Dynamic Array 는 size를 미리 고민할 필요가 없다는 장점이 있습니다.  
resizing 을 하는 방법은 여러가지가 있는데, 대표적으로 기존 Array size 의 2배 size 를 할당하는 doubling 이 있습니다. 

## resize를 하는 방식 ?
1. 원래 있던 배열보다 더 큰 사이즈 배열을 선언하고 원래 있던 배열 원소를 더 큰 사이즈 배열로 옮긴다.
2. 기존 배열은 메모리에서 삭제한다.

## Doubling ?
resize 의 대표적인 방법으로 데이터를 추가 (append O(1)) 하다가 `메모리를 초과하게 되면 기존 배열의 size 보다 두배 더 큰 배열을 선언하고 데이터를 일일이 옮기는(n개의 데이터를 일일이 옮겨야 하므로 O(n)) 방법입니다.`

## 데이터 추가(append)할 때의 시간 복잡도 
append의 총 과정을 살펴보면 데이터를 마지막 인덱스에 추가하는 O(1) 작업이 대다수이고, size를 넘어설 때는 size를 두 배 늘리고 데이터를 일일이 옮기는 과정 resize O(n) 이 아주 가끔 발생합니다. 결론부터 말하자면 append 의 전체적인 시간 복잡도는 O(1) 입니다. 좀 더 정확히 말하자면 amortized O(1)이라고 부릅니다.  

쉽게 설명하면 `가끔 발생하는 O(n)의 resize 하는 시간을, 자주 발생하는 O(1)의 작업들이 분담해서 나눠 가짐으로써 전체적으로 O(1)의 시간이 걸립니다.`

## Dynamic Array를 Linked List 와 비교하여 장단점을 설명해주세요
### Linked List와 비교했을 때 Dynamic Array의 장점은 ? 
- 데이터 접근과 할당이 O(1)로 굉장히 빠릅니다. 이는 index 접근하는 방법이 산술적인 연산 `[배열 첫 data 주소값] + [offset]으로 이루어져있기 때문입니다.(random access)`
- Dynamic Array의 `맨 뒤에 데이터를 추가하거나 삭제하는 것이 상대적으로 빠릅니다. O(1)`

### Linekd List와 비교했을 때 Dynamic Array의 단점은 ?
- Dynamic Array의 맨 끝이 아닌 곳에 data를 insert or remove 할 때, 느린 편입니다. O(n)  
느린 이유는 메모리상에서 연속적으로 데이터들이 저장되어 있기 때문에, `데이터를 추가 삭제할 때 뒤에 있는 data 들을 모두 한칸씩 shift 해야되기 때문입니다.`
- resize를 해야할 때, 예상치 못하게 `현저히 낮은 performance가 발생합니다.`
- resize에 시간이 많이 걸리므로 필요한 것 이상 memory 공간을 할당받습니다. 따라서 `사용하지 않고 있는 낭비되는 메모리 공간이 발생합니다.`