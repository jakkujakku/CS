# 스택(Stack)
1. 스택

## 스택
스택은 LIFO(Last In First Out)의 특징을 가지는 자료구조입니다. 스택은 자료를 저장할 때, 먼저 넣은 데이터를 가장 마지막에 꺼내게 됩니다

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5pe7v%2FbtrnJAxzW2k%2F0OFEzfLEK2nwpiekv1RrS0%2Fimg.png">

## 뼈대 만들기
```swift
struct Stack<T> {
    var elements: [T] = []
    
    var count : Int {
        return elements.count
    }
    var isEmpty : Bool {
        return elements.isEmpty
    }
}
```

## 삽입(push)
```swift
  mutating func push(_ element: T) {
        elements.append(element)
    }
```

## 조회(top)
```swift
    func top() -> T? {
        return elements.last
    }
```

## 삭제(pop)
removeLast는 배열이 비어있을 때 사용하면 에러를 발생시키지만, popLast를 사용하면 배열이 비어있을 때는 nil을 반환합니다.
```swift
    mutating func pop() -> T? {
        return elements.popLast()
    }
```

## 참고 
1. 참고 주소 : https://jeonyeohun.tistory.com/319