# ✅ 목록
1. Operation과 OperationQueue
2. Operation 1 : 기본
3. Operation 2 : 심화
4. OperationQueue
5. Custom Operation

## Operation과 OperationQueue
수행해야할 코드 블록들을 객체화 장점
- 재사용에 용이하다.
- 타입 간의 관계를 만들어줄 수 있다.
- 다양한 프로퍼티를 활용해볼 수 있다.
- 스케쥴링에 용이하다.

Operation은 GCD를 객체 지향적으로 재탄생시킨 것이라고 했습니다. Operation을 사용하면 동시성 프로그래밍과 관련된 모든 작업들은 Operation이라는 객체로서 만들어지게 됩니다.

이렇게 만들어진 Operation 객체들은 각각을 직접 실행시킬 수도 있지만 GCD처럼 OperationQueue라는 Queue에 넣어서 실행 및 관리해줄 수도 있습니다.

```swift
import Foundation

let order1 = BlockOperation {
    print("돈까스")
    print("떡볶이")
}

let order2 = BlockOperation {
    print("튀김 우동")
}

let order3 = BlockOperation {
    print("알리오 올리오")
    print("생맥주 2")
}

let order4 = BlockOperation {
    print("과일 세트")
    print("LA 갈비")
}

let order5 = BlockOperation {
    print("김치전")
    print("바닐라 아이스크림")
}

let orderBar = OperationQueue()
orderBar.maxConcurrentOperationCount = 2

orderBar.addOperation(order1)
orderBar.addOperation(order2)
orderBar.addOperation(order3)
orderBar.addOperation(order4)
orderBar.addOperation(order5)

// orderBar.addOperations([order1, order2, order3, order4, order5], waitUntilFinished: true)
```

## Operation 1 : 기본
### Operation

```swift
class Operation : NSObject
```
Operation은 추상클래스입니다. 
그러니 항상 이를 상속받는 타입을 사용해야합니다. 
하위 클래스는 커스텀 클래스로 직접 만들어주는 방법이 있고, 앞서 살펴본 코드처럼 BlockOperation이라는 하위 클래스를 사용하는 방법이 있습니다. 

### Operation 만들기

```swift
class BlockOperation : Operation
```

Operation의 하위 클래스인 BlockOperation을 활용하여 이렇게 Operation을 만들어줄 수 있습니다. 

```swift
let operation = BlockOperation {
    // some code
}

// BlockOperation의 메서드
operation.addExecutionBlock {
    // some code to be executed after the operation operation.
}

// Operation의 프로퍼티
operation.completionBlock = {
    // some code to be executed after the operation and executionBlocks
}
```

Operation을 이렇게 만들어서 Queue에 넣거나, 직접 실행시켜줄 수 있습니다. 

그런데 그 아래에 addExecutionBlock(() -> Void) 라는 메서드가 등장합니다. 
이 메서드를 호출하면 Operation의 동작이 끝나고 난 후에 원하는 코드를 실행해줄 수 있습니다. 
그리고 completionBlock이라는 Operation의 프로퍼티에 코드블록을 할당하면 Operation과 그와 연관된 executionBlock들이 모두 실행된 다음에 실행시킬 수 있습니다.

### Operation 실행하기
Opertion을 실행하는 방법은 크게 2가지 입니다. 
직접 실행하는 방법과 OperationQueue에 넣는 방법입니다.

```swift
// 직접 실행하기
operation.start()

// OperationQueue로 실행하기
let operationQueue = OperationQueue()
operationQueue.addOperation(operation)
```

OperationQueue에는 Operation을 넣어주기만 하면 바로 실행이 됩니다.

### Operation을 실행하면 어떤 스레드에서 실행될까?

> 기본적으로 start() 메서드를 통해 직접 실행하면 Operation을 실행한 현재 스레드에서 Operation을 실행하게 됩니다. 
이 내용은 Operation이 동기냐, 비동기냐에 따라 조금 차이가 있는데, 비동기 Operation의 경우에는 새로운 스레드를 만들어서 Operation을 처리합니다. 
그리고 Operation을 OperationQueue에 넣어서 실행하는 경우에는 Operation을 각각 새로운 스레드를 만들어 작업합니다.

### Operation의 프로퍼티
Operation에는 해당 Operation의 상태를 추적할 수 있는 다양한 프로퍼티가 존재합니다. 
GCD와 차별화되는 지점 중 하나임...DispatchWorkItem에는 isCancelled는 구현되어 있음.

아래의 Bool 타입 프로퍼티를 통해 해당 Operation이 현재 실행 중인지, 
취소되었는지, 대기 상태인지, 작업이 종료되었는지, Concurrent인지, Asynchronous인지 등을 알 수 있습니다. 
또한 name이라는 String 값을 확인할 수도 있지요. 
해당 프로퍼티들은 name을 제외하고는 모두 읽기 전용 프로퍼티입니다.

```swift
var isCancelled: Bool
var isExecuting: Bool
var isFinished: Bool
var isConcurrent: Bool
var isAsynchronous: Bool
var isReady: Bool
var name: String?
```

상태를 추적하는 프로퍼티들은 Operation의 상태에 따라 자동으로 값을 변경해줍니다. 
따라서 우리는 필요할 때 값을 확인하기만 하면 됩니다. 

### isConcurrent, isAsynchronous는 무엇인가요?
공식문서에 따르면 isConcurrent 대신 isAsynchronous 값을 사용하라고 합니다. 
macOS 10.6 이후의 버전에서는 사용되지 않는 프로퍼티로 간주하면 됩니다. 


### Operation의 상태 변화
Operation은 아래 그림과 같은 상태 주기를 가지고 있습니다.

<img src="https://user-images.githubusercontent.com/73867548/153431140-fe224946-dbe3-4167-8569-0543a653308b.png">

- isReady: Operation이 실행할 준비를 마치면 isReady 상태가 됩니다.
- isExecuting: Start가 호출된 후(혹은 OperationQueue에 의해 실행된 후) isExecuting 상태가 됩니다.
- isCancel: Operation을 cancel하게 되면 isCancel 상태가 됩니다.
- isFinished: cancel하지 않고 동작을 모두 마쳤다면 isFinished 상태가 됩니다.

## Operation 2 : 심화
### Cancel

Operation의 상태 중에서는 isCancelled 라는 프로퍼티가 있었습니다. 
네이밍으로 눈치를 채보면 Operation의 실행이 취소되었다는 말임. 

```swift
operation.cancel()
```

Operation은 cancel()이라는 메서드를 가지고 있습니다. 
이 메서드를 호출하면 해당 Operation의 상태를 isCancelled 상태로 만들어줄 수가 있습니다.
우리가 기대하는 것과는 다르게 cancel() 메서드는 현재 실행 중인 Operation의 동작을 직접적으로 (강제로) 취소시켜주지는 않습니다.

사실 cancel 메서드는 Bool 타입인 isCancelled 프로퍼티의 값만 True로 변경해주는 메서드입니다. 
즉, 실질적으로 코드의 동작을 중단시키는 메서드가 아닙니다. 
그러므로 Operation의 동작을 취소해주고 싶다면 cancel()을 호출한 후, 상태의 변경을 나타내는 isCancelled라는 프로퍼티를 추적하는 것을 통해서 작업을 관리해주면 됩니다.

### isAsynchronous

그리고 isAsynchronous라는 프로퍼티도 있었습니다. 
Operation이 Asynchronous로 동작할 것인가, Synchronous로 동작할 것인가에 대한 프로퍼티입니다. 

먼저 직접 실행 시켜줄 경우, 
Operation이 동기(synchronous)일 때는 Operation을 실행한 현재의 스레드에서 작업을 처리하게 되고, 
비동기(asynchronous)일 때에는 새로운 스레드를 만들어서 작업을 처리하게 됩니다.

그런데 만약 OperationQueue에 넣어서 작업을 처리하게 하면 isAsynchronous의 값과는 상관없이 모두 새로운 스레드를 만들어 작업을 처리합니다.

Operation의 isAsynchronous 프로퍼티의 기본값은 false입니다. 
즉 아무런 설정을 해주지 않았다면 동기(synchronous)로 동작한다는 뜻이지요. 
만약 비동기(asynchronous)로 동작하는 Operation이 필요하다면 Custom하여 Operation을 만들어주어야 합니다. 
isAsynchronous 프로퍼티는 읽기 전용이기 때문에 override를 통해 값을 설정해줄 수 있겠습니다.

OperationQueue를 사용한다면 Operation의 동기/비동기 상태와는 상관없이 모두 새로운 스레드에서 작업을 하게 됩니다.
그러므로 OperationQueue를 사용하여 Operation들을 처리한다면 isAsynchronous 상태 값을 전혀 신경쓰지 않으면서 효율적으로 Operation을 관리할 수 있습니다. 

### Dependency

종속성은 Operation 인스턴스들 사이에 종속적인 관계를 만들어서 실행의 순서를 관리하도록 해줍니다. 
기본적으로 Operation은 직접 실행한다면 실행한 순서대로, OperationQueue에 넣는다면 넣은 순서대로 작업을 처리하게 되는데요. 
addDependency(_:) 메서드로 종속적인 관계를 만들어주면 해당 Operation은 자신이 종속된 Operation의 작업이 끝날 때까지 실행할 수 없게 됩니다. 
즉 자신이 종속된 Operation의 작업이 무조건 우선시되어야 하는 것입니다.

```swift
import Foundation

let yagom = BlockOperation {
    print("🐻🐻🐻🐻🐻")
}

let noroo = BlockOperation {
    print("🦌🦌🦌🦌🦌")
}

let odong = BlockOperation {
    print("🌳🌳🌳🌳🌳")
}
```

odong.addDependency(yagom)
이러한 Operation을 3개 만들었다고 해봅시다. 
그리고 odong은 addDependency(_:)를 호출하여 yagom에 종속되었습니다.

```swift
odong.start() 
// error: Thread 1: "*** -[NSBlockOperation start]: receiver is not yet ready to execute"
```

odong은 yagom의 실행이 끝날 때까지 실행될 수 없기 때문입니다. 
yagom에 종속되었기 때문이지요. 
odong은 yagom의 실행이 끝나야 비로소 isReady 상태가 됩니다.

<img src="https://user-images.githubusercontent.com/73867548/153464440-ddedd889-2585-4000-a920-21a826cb7cc6.png">

```swift
yagom.start()
odong.start()

/* 출력
🐻🐻🐻🐻🐻
🌳🌳🌳🌳🌳
*/
```

이렇게 종속성을 어기지 않는 순서로 Operation을 실행시켜주면 정상적으로 잘 동작하는 것을 확인할 수 있습니다. 
재밌고 아주 강력한 기능이네요 👏 종속성은 Operation이 서로 다른 OperationQueue에 있더라도 유효합니다.

종속성은 addDependency(_:)로 추가해주고, removeDependency(_:) 메서드로 제거해줄 수 있습니다. 
또한 하나가 아닌 여러 개의 Operation에 대해서도 종속이 가능합니다. 
종속성을 잘못 관리하게 되면 런타임 에러가 발생하므로 섬세하게 다루어줄 필요가 있습니다. 
Operation이 많아지고, 관계가 복잡해진다면 더더욱 곤란한 상황이 있을 수 있습니다.

또한 종속성을 설정할 때에는 Deadlock을 조심해야합니다. 
교착 상태(Deadlock)란 실행 순서를 서로 기다리면서 결국 아무런 동작도 하지 못하는 상태를 말합니다. 


```swift
odong.addDependency(yagom)
yagom.addDependency(noroo)
noroo.addDependency(odong)
```

<img src="https://user-images.githubusercontent.com/73867548/153464994-2f3196b9-f233-4602-b935-5e41ec2c127a.png">

### 그 외

#### QoS (Quality of Service)
Operation 별로 각각의 QoS를 설정해줄 수 있습니다. 

#### waitUntilFinished()
waitUntilFinished() 메서드는 Operation이 동작하고 있는 스레드에서 해당 Operation이 끝날 때까지 기다리는 메서드입니다. 
이 메서드를 사용할 때에는 교착 상태(deadlock)에 빠지지 않도록 주의해야함.

## OperationQueue
### OperationQueue

Operation같은 경우는 좀 더 객체지향적으로 설계되어 sync/async에 대한 정보는 Operation이 가지고 있으며, 스레드 관리는 OperationQueue가 하는 식으로 역할이 분리되어 있었습니다. 
그리고 OperationQueue에서는 모든 Operation에 대해 새로운 스레드를 만들어서 작업을 처리하지요.

오퍼레이션 큐는 Operation을 호출합니다. 
operation을 큐에 추가하면 실행될 때까지 operation은 큐에 남아있게 되며, 추가한 operation을 직접 삭제할 수는 없습니다. 
그리고 모든 operation이 끝날 때까지 Queue는 유지되는데요, 이때 완료되지 않은 작업을 대기열에 중단시키면 Queue와 Operation이 메모리에 그대로 남아있는, 메모리 누수가 발생할 수 있다는 점을 주의하도록 합시다.

### main, current
```swift
class var main: OperationQueue
class var current: OperationQueue?
```

current는 현재 작업을 시작한 OperationQueue를 반환합니다. 
대기열을 확인할 수 없는 경우에는 nil이 될 수 있기 때문에 옵셔널 타입으로 정의되어 있습니다. 
main은 메인 스레드에서 동작하는 OperationQueue를 반환합니다. 
메인 스레드에서 작업해야할 때 사용하는 프로퍼티입니다.

```swift
let queue = OperationQueue()
let mainQueue = OperationQueue.main
let currentQueue = OperationQueue.current
```

### addOperation
```swift
func addOperation(_ op: Operation)
func addOperations(_ ops: [Operation], waitUntilFinished wait: Bool)
func addOperation(_ block: @escaping () -> Void)
```

addOperation은 OperationQueue에 Operation을 추가하는 메서드입니다. 
추가되는 동시에 Operation의 작업이 수행되지요. 
Operation을 Queue에 넣어 작업을 시작하는 메서드임.
파라미터에 따라 하나의 Operation, 다수의 Operation, 클로져 등 다양한 형태로 Queue에 추가할 수 있습니다.

```swift
let someOperation = BlockOperation {
    // some code
}

let operationQueue = OperationQueue()
operationQueue.addOperation(someOperation)
```

### maxConcurrentOperationCount

```swift
var maxConcurrentOperationCount: Int
```
maxConcurrentOperationCount는 한 번에 최대로 수행되는 작업의 갯수입니다. 
별도의 설정이 없는 기본적인 OperationQueue에 Operation이 들어오게 되면 기본적으로 비동기적으로 작업이 진행되는데요, 
하지만 이때 maxConcurrentOperationCount를 설정해주면 작업 순서에 맞게 동시에 작업될 수 있는 Operation의 수를 제한해줄 수 있습니다. 

```swift
import Foundation

let order1 = BlockOperation {
    print("돈까스")
    print("떡볶이")
}

let order2 = BlockOperation {
    print("튀김 우동")
}

... (중략)

let orderBar = OperationQueue()
orderBar.maxConcurrentOperationCount = 2

orderBar.addOperations([order1, order2, order3, order4, order5], waitUntilFinished: true)
```

### cancelAllOperations()
대기열에 있는 모든 Operation의 cancel() 메서드를 호출합니다. 
이때 역시 현재 실행 중인 Operation을 즉각적으로 종료하거나 대기열에서 Operation을 삭제하거나 하지는 않습니다.

### waitUntilAllOperationsAreFinished()
현재 스레드를 차단하고 OperationQueue의 모든 Operation 실행이 끝날 때까지 기다리는 메서드입니다. 
이 시간 동안 현재 스레드는 큐에 작업을 추가할 수 없지만 다른 스레드는 작업을 추가할 수 있습니다. 
DispatchGroup의 wait()와 비슷한 메서드임.

### qualityOfService
```swift
var qualityOfService: QualityOfService
```
Operation과 DispatchQueue에서 언급했던 QoS입니다. 
각각의 Operation에도 QoS를 설정해줄 수 있었는데 OperationQueue에도 QoS를 설정해줄 수 있습니다. 
대기열 작업을 효율적으로 수행할 수 있도록 여러 우선순위 옵션을 제공

### isSuspended
```swift
var isSuspended: Bool { get set }
```
isSuspended는 대기열이 Operation 스케줄링이 진행 중인지에 대한 상태를 나타내는 Bool 값입니다. 
false인 경우, 대기열의 Operation을 실행시키고, true인 경우, 대기열의 Operation을 실행하지는 않지만 현재 실행 중인 Operation은 계속 실행이 됩니다. 
중단된 대기열에 Operation을 추가할 수는 있지만 isSuspended가 false가 될때까지 실행되지는 않습니다.

Operation은 실행이 완료되어야만 OperationQueue에서 제거가 되는데, 대기열이 중단된 상태라면 Operation이 완료, 제거되지 못하고 계속 남아있는 상태가 됩니다. 

## Custom Operation
Operation을 직접 실행시키는 것은 권장하지 않는다는 이야기가 있었습니다. 
그 이유는 OperationQueue 사용하면 쉬운데, 굳이 비동기(aysnchronous) Operation을 만들어주면서 사용하는 것이 그렇게 효율적이지 않다는 점 때문입니다. 
그 외에 동기(synchronous) Operation을 실행하는 경우에는 스레드를 차단하게 된다는 단점도 있습니다.

### main()과 start()
<img src="https://user-images.githubusercontent.com/73867548/153516170-25d3236f-31bc-484b-8c80-df646d772e20.png">

결론부터 이야기하자면 메서드의 역할 분리 때문이라고 할 수 있습니다. 
main()에는 Operation이 실질적으로 수행해야하는 코드들이 담겨있습니다. 
start()에는 Operation이 실행되었을 때 필요한 상태 변화에 대한 코드와 실질적인 동작을 수행하는 main() 메서드를 호출하고 있습니다.

OperationQueue에서 Operation을 실행시킬 때에는 start() 메서드를 호출해줍니다. 
main()을 private으로 설정하지 않은 이유는 상속을 통한 재정의가 가능해야하기 때문

Operation을 Custom할 경우 Operation이 동기인지, 비동기인지에 따라 재정의해야하는 메서드가 다릅니다. 
공식문서에서도 main()은 non-concurrent(Synchronous) Operation에 대해 실행되고, 
start()는 concurrent(Asynchronous) Operation에 대해 실행된다고 합니다. 
(Operation에서 isConcurrent는 isAsynchronous로 대체하여 사용된다고 했습니다.)

### Custom Synchronous Operation

동기 Operation을 Custom 하는 방법은 매우 간단합니다. main() 메서드만 재정의해주면 됩니다. 

```swift
class MySyncOperation: Operation {
    let str: String
    let count: UInt

    init(str: String, count: UInt) {
        self.str = str
        self.count = count
    }

    override func main() {
        guard count > 1 else {
            print("count에는 1보다 큰 숫자를 입력해주세요.")
            return
        }

        for _ in 0 ... count - 1 {
            print(str)
        }
    }
}

let syncOperation1 = MySyncOperation(str: "오동나무", count: 3)
syncOperation1.start()

/* 출력
오동나무
오동나무
오동나무
*/
```

Operation의 실질적인 동작을 담당하는 main() 메서드만 재정의 해주고, 그 외에 필요한 코드를 구성하여 Custom Operation을 만들 수 있습니다. 

### Custom Asynchronous Operation

비동기 Operation을 Custom하는 경우는 동기 Operation을 Custom 하는 경우보다 고려해야할 사항이 더 많습니다. 
상태 변화를 담당하고 있는 start() 메서드를 재정의해야하기 때문입니다. 

그러나 main() 재정의하여 비동기 작업을 해주는 경우, Operation이 finished 상태로 변하는 시점이 부정확해지는 곤란한 상황이 발생합니다. 

<img src="https://user-images.githubusercontent.com/73867548/153431140-fe224946-dbe3-4167-8569-0543a653308b.png">

먼저 start()를 재정의하지 않고 main()만 재정의하는 경우 어떤 일이 발생하는지 확인해봅시다.

```swift
// 문제의 코드
class StrangeAsyncOperation: Operation {
    let str: String
    let count: UInt

    init(str: String, count: UInt) {
        self.str = str
        self.count = count
    }

    override func main() {
        DispatchQueue.global().async { [str, count] in
            guard count > 1 else {
                print("count에는 1보다 큰 숫자를 입력해주세요.")
                return
            }

            for _ in 0 ... count - 1 {
                print(str)
            }
        }
    }
}

let strangeOperation = StrangeAsyncOperation(str: "오동나무", count: 5)
strangeOperation.completionBlock = {
    if strangeOperation.isFinished {
        print("Finished!")
    }
}
strangeOperation.start()

/* 출력
Finished!
오동나무
오동나무
오동나무
오동나무
오동나무
*/
```

global().async에서 비동기 작업을 처리해주니 비동기 작업이 끝나기도 전에 Operation의 상태가 Finished로 변경되었기 때문입니다. 
그러므로 상태 변경에 대한 코드도 비동기 작업에 맞게 다시 재정의가 되어야하는 것이지요. 
먼저 비동기 Operation의 부모 클래스가 될 AsyncOperation입니다.

```swift
class AsyncOperation: Operation {
    // 1
    enum State: String {
        case ready, executing, finished

        // 2
        var keyPath: String {
            "is\(rawValue.capitalized)"
        }
    }

    // 3
    var state = State.ready {
        // 4
        willSet {
            willChangeValue(forKey: state.keyPath)
            willChangeValue(forKey: newValue.keyPath)
        }
        didSet {
            didChangeValue(forKey: oldValue.keyPath)
            didChangeValue(forKey: state.keyPath)
        }
    }

    // 5
    override var isExecuting: Bool {
        state == .executing
    }
    override var isFinished: Bool {
        state == .finished
    }
    override var isAsynchronous: Bool {
        true
    }
    // 6
    override func start() {
        if isCancelled {
            state = .finished
            return
        }
        state = .executing
        main()
    }

    override func cancel() {
        state = .finished
    }
}
```

타입 내부에 상태를 열거형으로 정의합니다. 
state라는 프로퍼티를 만들어 값을 업데이트하는 식으로 구현하기 위함입니다.
Operation에서는 읽기 전용인 프로퍼티들을 업데이트하기 위해 KVO로 프로퍼티에 접근해야합니다. 
그에 필요한 String 타입의 keyPath 값을 구현합니다.
Operation의 상태에 대한 프로퍼티를 변수로 만들어줍니다. 
state 변수를 기준으로 Operation의 상태 프로퍼티 값을 변경합니다.
KVO로 상태 프로퍼티 값을 갱신시켜주기 위해 willSet과 didSet을 구현합니다.
isExecuting, isFinished, isAsynchronous의 값을 재정의해줍니다.
start() 메서드를 재정의 해줍니다. 내부에서 isCancelled 상태값을 확인해준 후, state를 변경하고 main()을 호출해줍니다. (main 메서드는 다시 한 번 상속을 통해 재정의 될 것입니다.)

```swift
class MyAsyncOperation: AsyncOperation {
    let str: String
    let count: UInt

    init(str: String, count: UInt) {
        self.str = str
        self.count = count
    }

    override func main() {
        DispatchQueue.global().async { [str, count] in
            guard count > 1 else {
                print("count에는 1보다 큰 숫자를 입력해주세요.")
                return
            }

            for _ in 0 ... count - 1 {
                print(str)
            }

            // 7
            self.state = .finished
        }
    }
}

let strangeOperation = MyAsyncOperation(str: "오동나무", count: 5)
strangeOperation.completionBlock = {
    if strangeOperation.isFinished {
        print("Finished!")
    }
}
strangeOperation.start()

/*
오동나무
오동나무
오동나무
오동나무
오동나무
Finished!
*/
```

main() 메서드의 동작이 끝나는 시점에 state를 finished로 변경해줍니다. 

start() 메서드를 재정의할 때에는 super를 호출하면 안됩니다. 
Operation의 상태를 변경하는 동작이 의도와 다르게 동작할 수 있기 때문입니다. 
또한 start() 메서드를 재정의하면서 isCancelled 프로퍼티를 확인하는 작업도 놓치지 않도록 합시다.

## 참고 
1. 참고 자료 : https://yagom.net/courses/동시성-프로그래밍-concurrency-programming/lessons/operation/