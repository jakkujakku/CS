# ✅ 목차
1. 링크드 리스트
2. 추가
3. 삽입
4. 삭제
## 링크드 리스트 (Linked List)
- 링크드 리스트는 데이터를 가지고 있는 노드들을 포인터로 선형적으로 연결해 사용하는 자료구조입니다. 삭제와 삽입에 O(1)의 시간복잡도가 걸리기 때문에(원하는 노드까지의 탐색시간 제외) 빈번하게 삽입과 삭제가 필요한 상황에 유용하게 사용할 수 있습니다. 
하지만 동시에 배열처럼 인덱스를 통한 Random Access가 불가능하고, 처음부터 원하는 데이터가 나올 때까지 탐색을 진행해야하기 때문에 탐색에는 O(N)의 시간복잡도가 요구된다는 단점이 있습니다.

## 뼈대 만들기
```swift
import Foundation
 
class Node<T: Equatable> {
    let id: Int
    let data: T
    var next: Node?
    
    init(id: Int, data: T, next: Node? = nil) {
        self.id = id
        self.data = data
        self.next = next
    }
}
```

## Linked List
```swift
struct LinkedList<T: Equatable> {
    var head: Node<T>?
    var tail: Node<T>?
   
    func showList() {
        var now = head
        print("===== Linked List ======")
        while now != nil {
            now?.next == nil
            ? print("id: \(now?.id) | data: \(now?.data)")
            : print("id: \(now?.id) | data: \(now?.data) -> ")
            now = now?.next
        }
        print("========================")
    }
}
```
- 링크드 리스트의 프로퍼티에는 가장 앞에 위치한 노드를 가리키는 head와 가장 마지막 노드를 가리키는 tail로 구성됩니다. 

## 추가(add) - O(1)
```swift
mutating func add(node: Node<T>) {
        // head node does not exist
        if head == nil {
            head = node
            tail = node
            return
        }
        
        // search for last node and attatch new
        tail?.next = node
        tail = node
    }
```
- 추가 연산은 두 가지 경우가 있을 수 있습니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbdS0Kg%2FbtrnC3Oz53H%2F6cmfRxj6khNXkoYIWog7K0%2Fimg.png">
- 링크드 리스트에 아무 노드도 없을 경우인데요, 이때는 head가 가리키는 노드가 없으니 초기값인 nil이 들어있겠죠? 그래서 head의 값을 확인해보고 nil이라면 head에 새로 들어온 노드를 연결해주고, tail도 nil일테니 여기에도 새로운 노드를 연결해줍니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXyRWB%2FbtrnDFUc8ax%2FDfiLruqekfqkG8u5I5acz1%2Fimg.png">
- 이미 링크드 리스트에 데이터들이 들어있는 경우입니다. 이때는 가장 마지막 노드를 가리키는 tail 포인터로 찾아가서 tail이 가리키는 노드의 다음노드에 새 노드를 연결해준 뒤, tail이 가리키는 노드를 새로운 노드로 변경해줍니다.

## 탐색(search) - O(N)
- 링크드 리스트는 Random Access가 불가능해서 특정한 노드에 접근하려면 항상 제일 처음 노드인 head 부터 원하는 노드를 찾을 때까지 next 프로퍼티를 통해 다음노드를 재귀적으로 탐색해야합니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5X6rz%2FbtrnLIhQg51%2FF60yNQUFW4GOzQWBbgRtJ1%2Fimg.png">
- 이렇게 노드가 중간에 있는 노드까지 탐색을 하고싶다고 해보겠습니다. 찾고싶은 노드의 id는 4이네요.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fyex4m%2FbtrnGHDGAPI%2FV1SgQJM1W2aasRwAvenlkK%2Fimg.png">
- 탐색을 시작하기 위해 head 와 tail이 아닌 커서의 역할을 할 포인터 now를 만들어주겠습니다
```swift
now = now.next
```
- id를 발견한다면 해당 노드의 참조 값을 그대로 반환하고, id를 찾지 못한다면 마지막 노드까지 이동해 nil을 반환하게 하면 됩니다.
```swift
func searchNode(with data: T) -> Node<T>? {
        var now = head
        while now?.data != data && now != nil {
            now = now?.next
        }
        return now
    }
```
## 삽입(insert) - O(1) or O(N)
- 삽입 연산은 가장 뒤가 아닌 어떤 특정한 노드의 앞이나 뒤에 새로운 노드를 끼워넣는 것을 의미하는데요, 삽입을 하려면 원하는 지점까지의 탐색이 선행되어야 합니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb1iNxh%2FbtrnDpjf41p%2FM3HABiVc5WD0j03MiWG3MK%2Fimg.png">
탐색에서의 예시를 그대로 사용해서 id가 4인 노드를 찾았다고 해보겠습니다. 여기서 목표는 id가 4인 노드와 id가 5인 노드 사이에 새로운 노드를 삽입하는 것입니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEmpaZ%2FbtrnEJPjWJG%2F4bBajzG4RTe1q4nD7LBhKk%2Fimg.png">
새로운 노드의 삽입은 노드들 사이의 포인터만 바꾸면 쉽게 가능할 것 같습니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcT01Az%2FbtrnLHcaxYJ%2Fv6ECU8Y65ZNB5p6o1DkCA1%2Fimg.png">
방금 찾은 노드가 가리키던 다음 노드를 새로운 노드 또한 가리키게 만들고, 
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbssFu4%2FbtrnGkWmOQi%2FUrBPNsETD8msOQ2h1GD8n1%2Fimg.png">
찾은 노드가 이제 새로 삽입할 노드를 가리키게 하면 간단하게 삽입 연산이 완성됩니다. 목표 노드를 찾기 위해서 탐색연산을 수행하기 때문에 O(N)의 시간복잡도가 먼저 요구되고, 그 이후에 삽입은 포인터의 변경만 있기 때문에 O(1)에 끝나게 됩니다.
 
O(1)로 삽입이 되는 경우도 있는데요, head나 tail 앞이나 뒤에 새로운 노드를 삽입하면 탐색이 필요없기 때문에 O(1)에 삽입이 가능합니다.

```swift
 mutating func insert(node: Node<T>, after id: Int) {
        var now = head
        while now?.id != id && now?.next != nil {
            now = now?.next
        }
        
        node.next = now?.next
        now?.next = node
    }
    
    mutating func insert(node: Node<T>, before id: Int) {
        var now = head
        
        if now?.id == id {
            head = node
            node.next = now
            return
        }
        
        while now?.next?.id != id && now?.next != nil {
            now = now?.next
        }
        
        node.next = now?.next
        now?.next = node
    }
```
## 삭제(delete) - O(1) or O(N)
삭제 연산은 삽입과 유사합니다. 먼저 삭제할 노드를 찾고, 그 앞에 있는 노드의 next 포인터를 삭제할 노드의 다음 노드로 정해주면 됩니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHUepY%2FbtrnLIPF6H6%2F7esd0gnCSYVKWhKIfRGkKk%2Fimg.png">
이번에는 삭제할 노드까지 찾아가면 이전 노드에 접근할 방법이 없기 때문에 다음 노드가 삭제할 노드가 될 때까지 now 포인터를 움직여줍니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbfZS2t%2FbtrnB8bNqnk%2FCFwMtwhKQbTMMyyt4HHYnK%2Fimg.png">
그리고 삭제할 노드를 건너뛰고 그 다음 노드와 현채 포인터를 연결해줍니다.
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWvX4K%2FbtrnB8W9AiC%2FX2bfCRTsRyndqeYZfToNH0%2Fimg.png">
삭제 역시도 삽입처럼 탐색에 O(N), 삭제에 O(1)의 시간복잡도가 요구되기 때문에 삭제할 노드가 head 나 tail이 아닐 때는 O(N), head나 tail일 때는 O(1)이 시간복잡도가 됩니다.
```swift
mutating func delete(node: Node<T>) -> Bool {
        var now = self.head
        
        // if target node is at head
        if now === node {
            if self.head === self.tail { self.tail = nil }
            self.head = now?.next
            return true
        }
        
        while now?.next !== node && now?.next != nil {
            now = now?.next
        }
        
        // no matching node to delete
        if now == nil { return false }
        
        if now?.next === tail {
            tail = now
        }
        
        // delete node
        now?.next = now?.next?.next
        return true
    }
```

## 참고
관련 주소 : https://jeonyeohun.tistory.com/320