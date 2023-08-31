# 목차
1. 큐
2. 뼈대 만들기
3. 링크드 리스트
4. 삽입
5. 삭제
6. 참고 

## 큐(Queue)
- 큐는 FIFO(First In First Out)의 특성을 가지는 자료구조입니다. 이름처럼 가장 먼저 넣은 데이터를 가장 먼저 꺼내는 특징을 가지고 있습니다. 
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbzUuSi%2FbtrnHSecRlD%2FQKUEkY6DjWBRg2TY6KFJqk%2Fimg.png">

## 뼈대 만들기
```swift
var list = TwoPointerLinkedList<T>()
    
init(_ elements: [T] = []) {
    for element in elements {
        list.add(node: Node(data: element))
    }
}
    
var count : Int {
    return list.count
}
var isEmpty : Bool {
    return list.head == nil
}
var front: T? {
    return list.front?.data
}
var back: T? {
    return list.back?.data
}
```
TwoPointerLinkedList는 head, tail 두 개의 포인터로 링크드 리스트의 가장 앞과 뒤를 가리키게 하는 링크드 리스트 자료구조입니다.
 
- 생성자에서는 만약에 배열이 한 번에 들어오면 순서대로 링크드 리스트에 추가해주는 로직을 넣어두었고, 
- count 프로퍼티는 노드의 개수
- isEmpty 프로퍼티는 head 포인터가 비어있는지를 통해 값을 반환하도록 구성

## 링크드 리스트
```swift
import Foundation
 
struct TwoPointerLinkedList<T> {
    var head: Node<T>?
    var tail: Node<T>?
    var count: Int = 0
    
    var front: Node<T>? {
        return head
    }
    var back: Node<T>? {
        return tail
    }
    
    mutating func add(node: Node<T>) {
        if self.head == nil {
            self.head = node
            self.tail = node
        } else {
            self.tail?.next = node
            self.tail = node
        }
        self.count += 1
    }
    mutating func removeFirst() -> Node<T>? {
        guard head != nil else {
            self.clear()
            return nil
        }
        let deleted = self.head
        self.head = head?.next
        self.count -= 1
        
        if head == nil {
            self.clear()
        }
        return deleted
    }
    mutating func clear() {
        self.head = nil
        self.tail = nil
    }
}
```
기존 링크드 리스트와 거의 동일하지만, clear 메서드를 통해 모든 노드를 한번에 해제할 수 있게 하고, 노드의 개수를 탐색없이 확인할 수 있도록 하는 count 프로퍼티와 가장 앞 노드와 가장 뒤 노드를 쉽게 확인할 수 있는 front, back 프로퍼티를 추가

## 삽입(push) 
삽입 연산은 기존의 링크드 리스트의 추가 연산을 그대로 사용할 수 있습니다.
```swift
    mutating func push(_ element: T) {
        list.add(node: Node(data: element))
    }
```
만약, 큐 자료구조에서 자체적으로 만들어서 링크드 리스트에 주입하는 방식으로 구성한다면?
```swift
mutating func add(node: Node<T>) {
    if self.head == nil {
        self.head = node
        self.tail = node
    } else {
        self.tail?.next = node
        self.tail = node
    }
    self.count += 1
}
```
새로운 노드를 tail 뒤에 붙여주거나, 리스트가 비어있는 경우에는 추가해주는 로직과 노드의 개수를 늘려주는 로직이 들어가있습니다. 

## 삭제(pop)
삭제 연산은 링크드 리스트에서 가장 앞 노드를 반환하고 지우는 방식으로 구현
1. 리스트가 있을 때
<img src= "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FJSnFz%2FbtrnC316ZuJ%2Fu5xtrYaBMUZFXBi5aGQL4k%2Fimg.png">
2. 현재 head 가 가리키는 값을 변수에 임시로 저장해 메모리에서 해제되지 않도록 강한 참조
<img src= "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FciRkta%2FbtrnL9l8bP5%2FKxGUvMioZh6OY6YcKDAL01%2Fimg.png">
3. head 포인터를 한 칸 앞으로 이동시켜서 리스트에서 가장 앞에 있던 노드를 제외시켜줍니다. 그리고 임시로 저장했던 노드를 그대로 반환하면 pop 연산이 완성
<img src= "https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcuV1e9%2FbtrnCmtNpjM%2FivjwgZkrQgPrMgj3ddS2E0%2Fimg.png">

```swift
mutating func removeFirst() -> Node<T>? {
    guard head != nil else {
        self.clear()
        return nil
    }
    let deleted = self.head
    self.head = head?.next
    self.count -= 1
    
    if head == nil {
        self.clear()
    }
    return deleted
}
```

# 참고
1. 관련 주소 : https://jeonyeohun.tistory.com/321