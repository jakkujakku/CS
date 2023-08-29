# 목록 
1. DispatchQueue의 초기화
2. QOS
3. Async
4. CompletionHandler
5. DispatchGroup
6. Race Condition과 Thread Safe
7. DispatchSemaphore
8. UI 작업은 왜 Main Thread에서 해야할까요?

## DispatchQueue의 초기화
- DispatchQueue를 초기화하는 메서드는 아래와 같습니다. 
- label을 제외한 파라미터들에는 기본값이 설정되어 있기 때문에 커스텀이 필요한 파라미터에 대해서만 초기화를 해줄 수 있습니다. 

```swift
convenience init(label: String,
                 qos: DispatchQoS = .unspecified,
                 attributes: DispatchQueue.Attributes = [],
                 autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency = .inherit,
                 target: DispatchQueue? = nil)
```

1️. label
```swift
let myDispatchQueue = DispatchQueue(label: "odongnamu")
```
- 직관적으로 눈치채신 것처럼 DispatchQueue의 label을 설정해주는 파라미터입니다. 
- 이 label은 디버깅 환경에서 추적하기 위해서 작성하는 String 값입니다. 식별자와 같은 것 

2️. qos
- qos는 DispatchQoS 타입의 값을 받는 파라미터입니다. 
- 여기서 QoS란 Quality of Service의 약자로, 실행 될 Task들의 우선 순위를 정해주는 값입니다.

3️. attributes
- attributes는 DispatchQueue의 속성을 정해주는 값입니다. 
- .concurrent로 초기화한다면 다중 스레드 환경에서 코드를 처리하는 DispatchQueue가 되는 것입니다. global()처럼 :)
- 이 값을 빈 배열, 즉 기본 값으로 아무 설정을 하지 않는다면 우리는 Serial DispatchQueue를 만들게 됩니다.

.initiallyInactive라는 녀석입니다. 우리가 지금까지 사용했던 일반적인 DispatchQueue들은 sync나 async 등 코드 블록을 호출하는 즉시 DispatchQueue 작업이 처리되었습니다. 하지만 이 .initiallyInactive는 그 작업을 제어해줄 수 있지요. 즉 sync나 async를 호출하더라도 작업을 큐에 담아놓을 뿐, active()를 호출하기 전까지는 작업을 처리하지 않는 것입니다. 

```swift
import Foundation

let yellow = DispatchWorkItem {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

let myDispatch = DispatchQueue(label: "Odong", attributes: .initiallyInactive)

myDispatch.async(execute: yellow) // 코드 블록 호출 안됨.
myDispatch.activate()
```

4️. autoreleaseFrequency
- DispatchQueue가 자동으로 객체를 해제하는 빈도의 값을 결정하는 파라미터입니다. 
- 즉 객체를 autorealease해주는 빈도이며 기본값은 inherit입니다.
    - inherit: target과 같은 빈도를 가집니다.
    - workItem: workItem이 실행될 때마다 객체들을 해제합니다.
    - never: autorelease를 하지 않습니다.

5️. target
코드 블록을 실행할 큐를 target으로 설정할 수 있습니다.

## QOS
- QoS의 우선 순위는 이런식으로 무엇에 더 에너지를 많이 투자할까를 결정하는 것 
- 일이 처리되는 순서를 결정하는 요소는 아니지만 어느정도 영향을 미칠 수는 있음 
- 더 많은 에너지를 쏟는다는 것은 더 많은 스레드를 할당한다는 이야기

초기화 구문과 뒤에 등장할 async, sync의 파라미터로 DispatchQoS 우선 순위를 할당할 수 있습니다. 그리고 시스템은 QoS 정보를 통해 스케쥴링, CPU 및 I/O 처리량, 타이머 대기 시간 등의 우선 순위를 조정합니다. DispatchQoS는 열거형 타입으로, 총 6개의 QoS 클래스가 있으며 4개의 주요 유형와 다른 2 개의 특수 유형으로 구분이 가능합니다. (높은 순서대로, userInteractive, userInitiated, default, utility, background, unspecified 입니다.)

우선 순위가 높을 수록 더 많은 전력을 소모합니다. 그러므로 수행 작업에 적절한 QoS를 할당하면 앱이 더 반응적(responsive)이고 보다 효율적인 에너지 사용이 가능해집니다. 우선 순위가 높은 순서대로 나열하였습니다.

1️⃣ User-interactive
main thread에서 작업하며, 사용자 인터페이스 새로고침, 애니메이션 등 사용자와 상호작용하는 작업에 할당합니다. 작업이 빠르게 수행되지 않으면, 유저 인터페이스는 멈추게 됩니다. 반응성(responsiveness)과 성능(performance)에 중점을 둡니다.

2️⃣ User-initiated
문서를 열거나 버튼을 클릭해 액션을 수행하는 것처럼 빠른 결과를 요구하는 유저와의 상호작용 작업에 할당합니다. 몇 초 이내의 짧은 시간 내에 수행해야하는 작업으로 반응성과 성능에 중점을 둡니다.

3️⃣ Default
QoS를 할당해주지 않을 경우 기본값으로 사용되며 User Initiate와 Utility의 중간 수준의 레벨입니다.

4️⃣ Utility
데이터를 읽거나 다운로드하는 작업처럼 작업이 완료되는데에 어느정도 시간이 걸리거나 즉각적인 결과가 요구되지 않는 작업에 할당합니다. 반응성, 성능, 에너지 효율의 밸런스에 중점을 둡니다.

5️⃣ Background
index 생성, 동기화, 백업 등 사용자가 볼 수 없는, 백그라운드의 작업에 할당합니다. 에너지 효율에 중점을 둡니다.

6️⃣ Unspecified
Unspecified는 QoS의 정보가 없음을 나타내며, 시스템이 QoS를 추론해야 합니다.

## Async
우리는 async{ code block }혹은 async(execute:)로 비동기 작업을 처리해왔습니다. 
사실 async라는 메서드에는 우리가 사용한 클로저 외에도 기본값을 가지는 3가지의 파라미터가 더 존재합니다. 

```swift
func async(group: DispatchGroup? = nil, qos: DispatchQoS = .unspecified, flags: DispatchWorkItemFlags = [], execute work: @escaping () -> Void)
```

1️⃣ group
DispatchQueue의 async 코드 블록을 묶어서 관리해주는 DispatchGroup입니다. 여러 스레드에서 비동기로 작업을 처리하다보면 여러 개의 작업을 함께 관리해주어야할 때가 있습니다. 

2️⃣ qos
QoS는 이전 페이지에서 살펴보았던 DispatchQueue와 같은 내용입니다. 역시 적절한 케이스를 설정해주면 시스템이 알아서 관리해줍니다.

3️⃣ flags
DispatchWorkItemFlags 타입의 값을 받는 파라미터입니다. 코드 블록을 실행할 때 적용될 추가 속성을 결정합니다. 기본 값으로는 아무 속성도 부여하지 않습니다. flags 값은 OptionSet이므로 여러 가지의 속성을 한 번에 부여할 수도 있습니다. 

- assingCurrentContext: 코드 블록을 실행하는 context(Queue 혹은 스레드)의 속성을 상속받습니다. QoS와 같은 속성을 동일하게 한다는 이야기입니다.
- barrier: concurrnet queue 환경에서 barrier(장벽, 차단) 역할을 합니다. barrier 속성의 코드 블록이 실행되기 전에 실행되었던 코드들은 완료까지 실행되고, barrier 속성의 코드 블록이 실행되기 전까지 다른 코드 블록은 실행되지 않습니다.
- detached: 실행할 코드 블록에 실행 중인 context(Queue 혹은 스레드)의 속성을 적용하지 않습니다.
- enforceQoS: 실행 중인 context의 QoS보다 실행할 코드 블록의 QoS에 더 높은 우선 순위를 부여합니다.
- inheritQoS: enforceQoS와 반대로 실행 중인 context의 QoS에 더 높은 우선 순위를 부여합니다.
- noQoS: QoS를 할당하지 않고 코드 블록을 실행시킵니다. assingCurrentContext보다 우선시 되는 속성입니다.

## CompletionHandler
completionHandler 혹은 completion이라는 클로저를 가진 메서드들을 보신 적이 있을 것 입니다. 이러한 클로저들은 함수의 실행 순서를 보장받을 수 있는 클로저입니다. 특히 escaping 클로저는 함수의 실행이 끝나면 함수의 밖에서 실행되는 작업들이지요. 

<img src="https://user-images.githubusercontent.com/73867548/146380895-ffa22af8-f104-40b8-80d4-d59b5a79a8b5.png">

대표적인 예로 URLSession을 들 수 있습니다. 서버와의 통신을 도와주는 API인 URLSession은 서버에서 데이터를 받아오는 메서드를 비동기로 실행시킵니다. 하지만 비동기로 작업이 처리되는 경우, 우리는 그 작업이 언제 끝날지를 정확하게 파악할 수가 없습니다. 스레드는 시스템이 관리해주기 때문입니다. 그렇기 때문에 completionHandler 혹은 completion와 같은 클로저를 구현해준다면 작업이 끝나는 시점에 원하는 동작을 수행시켜 줄 수 있는 것입니다.

## DispatchGroup
```swift
class DispatchGroup : DispatchObject
```

DispatchGroup은 비동기적으로 처리되는 작업들을 그룹으로 묶어, 그룹 단위로 작업 상태를 추적할 수 있는 기능입니다. 그러니까 DispatchGroup을 사용하면 async들을 묶어서 그룹의 작업이 끝나는 시점을 추적하여 어떠한 동작을 수행시킬 수 있습니다. 이때 묶어줄 async 작업들이 꼭 같은 큐, 스레드에 있지 않더라도 묶어줄 수 있습니다.

DispatchGroup은 async에서만 사용할 수 있습니다. 비동기로 처리되는 작업은 작업이 끝나는 시점을 정확하게 예측할 수가 없습니다. 시스템이 관리해주기 때문이지요. 반면 동기로 처리되는 작업들은 끝나는 시점을 예측할 수 있습니다. 작업이 처리되기를 반드시 기다렸다가 다음 작업을 수행할 수 있기 때문이지요. 따라서 동기로 처리되는 경우에는 작업 종료 시점을 따로 추적할 필요가 없습니다.

### group에 등록하기: enter, leave
DispatchGroup은 특별한 초기화 구문이 없습니다. init() 메서드로 바로 초기화해 필요한 만큼 인스턴스를 만들어서 사용하면 됩니다. 함께 관리할 작업들에는 같은 인스턴스를 지정해주어야 합니다.

DispatchGroup을 사용하는 방법은 2가지가 있습니다.
– async를 호출하면서 파라미터로 group을 지정해줍니다.
– enter, leave를 코드의 앞뒤로 호출하여 group을 지정해줍니다.

enter와 leave는 DispatchGroup이 enter()부터 leave()까지 포함된다라는 의미입니다.

```swift
let group = DispatchGroup()

// enter, leave를 사용하지 않는 경우
DispatchQueue.main.async(group: group) {}
DispatchQueue.global().async(group: group) {}

// enter, leave를 사용하는 경우
group.enter()
DispatchQueue.main.async {}
DispatchQueue.global().async {}
group.leave()
```

이렇게 원하는 작업들을 group으로 묶어주었습니다. 이제 묶어낸 그룹에 대해 notify() 혹은 wait()으로 작업을 추적해줄 수 있습니다.

### notify
notify는 DispatchGroup의 업무 처리가 끝나는 시점에 원하는 동작을 수행하기 위한 메서드입니다.

```swift
import Foundation

let red = DispatchWorkItem {
    for _ in 1...5 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

let yellow = DispatchWorkItem {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

let blue = DispatchWorkItem {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

// group.enter()
// DispatchQueue.global().async(execute: blue)
// DispatchQueue.global().async(execute: red)
// group.leave()

group.notify(queue: .main) {
    print("모든 작업이 끝났습니다.")
}
```

이 코드를 실행하면 어떻게 되나요? notify 메서드에 의해 group의 모든 작업이 끝나기를 기다렸다가 코드 블록을 실행시켜줍니다. 이때 notify의 파라미터 queue는 코드블록을 실행시킬 queue를 말합니다.


### wait
wait는 DispatchGroup의 수행이 끝나기를 기다리기만 하는 메서드입니다. notify와 달리 별도의 코드 블록을 실행하지 않습니다. 따라서 코드 블록을 실행시킬 queue를 지정할 필요도 없지요.

```swift
let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: blue)
DispatchQueue.global().async(group: group, execute: red)

group.wait()
print("모든 작업이 끝났습니다.")

// group.wait(timeout: 10)
// print("모든 작업이 끝났습니다.")
```

wait 메서드에는 timeout 파라미터를 설정해줄 수 있습니다. 만약 timeout에 10을 전달하면 group을 딱 10초 동안만 기다리는 것입니다. 만약 10초가 넘어갔는데도 group의 작업이 끝나지 않는다면 더 이상 기다리지 않고 다음 코드를 실행해버립니다. (asyncAfter와 마찬가지로 wallTimeout을 파라미터로 받을 수도 있습니다.)

DispatchGroup을 사용하면 이러한 문제를 해결할 수도 있습니다.

```swift
import Foundation

let black = DispatchWorkItem {
    for _ in 1...3 {
        print("🖥🖥🖥🖥🖥")
        sleep(1)
    }
}

let white = DispatchWorkItem {
    for _ in 1...3 {
        print("📃📃📃📃📃")
        sleep(2)
    }

    DispatchQueue.global().async(execute: yellow)
}

let group = DispatchGroup()

DispatchQueue.global().async(group: group, execute: black)
DispatchQueue.global().async(group: group, execute: white)

group.wait()
print("모든 작업이 끝났습니다.")

/* 출력
🖥🖥🖥🖥🖥
📃📃📃📃📃
🖥🖥🖥🖥🖥
📃📃📃📃📃
🖥🖥🖥🖥🖥
📃📃📃📃📃
😀😀😀😀😀
모든 작업이 끝났습니다.
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
*/
```

black과 white 작업을 group으로 묶어서 wait()를 호출해주었지만 😀😀😀😀😀이 모두 출력되기 전에 다음 코드가 실행되었습니다. 비동기 작업 내부에서 파생된 비동기 작업에 대해서는 기다려주지 않은 것이지요. 

white의 작업은 😀😀😀😀😀를 또 다른 global 스레드에서 async로 처리하도록 DispatchQueue에 요구하고 있습니다. 즉 다른 스레드에 일을 넘기는 것으로 white의 작업은 끝난 것이지요.

따라서 이러한 문제는 😀😀😀😀😀를 호출하는 async도 같은 group으로 묶어주는 것으로 해결할 수 있습니다. 

## Race Condition과 Thread Safe
### Race Condition
```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("야곰: \(card) 카드를 뽑았습니다!")
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("노루: \(card) 카드를 뽑았습니다!")
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        let card = cards.removeFirst()
        print("오동나무: \(card) 카드를 뽑았습니다!")
    }
}
```

### Race Condition
- 여러 개의 스레드를 사용하지 않는 프로그래밍에서는 어떤 상황에서나 코드가 순서대로(step by step) 실행이 되지만 스레드가 여러 개인 상황에서는 코드가 동시에 실행되어 하나의 값에 동시에 접근하는 경우가 발생할 수 있습니다. 

### Thread Safe
Race Condition이 발생하는 이유는 Swift의 배열이 Thread Safe하지 않기 때문입니다.
Thread Safe하다는 것은 여러 스레드에서 동시에 접근이 불가능한 것을 말합니다. 
실제로 Swift의 대부분의 타입들은 Thread Unsafe합니다. 
Thread Unsafe하다는 말은 여러 스레드에서 동시에 접근이 가능한 상태라는 말입니다. 

## DispatchSemaphore
```swift
class DispatchSemaphore : DispatchObject
```
공유 자원에 접근할 수 있는 스레드의 수를 제어해주는 역할을 합니다. 
몇 개의 스레드에 접근을 허용할 것인지 제어할 수 있기 때문에 접근을 1개의 스레드만 허용한다면 Race Condition을 방지할 수 있는 것입니다.

DispatchSemaphore는 semaphore count를 카운트하는 식으로 동작합니다. 
예를 들어 하나의 스레드가 접근을 하면 count에 -1을, 접근이 끝나면 count에 +1을 해줍니다. 
따라서 설정해준 count만큼만 스레드가 접근할 수 있도록 관리해주는 것이지요. 
만약 허용된 스레드의 수만큼 접근된 상태라면 다른 스레드는 접근하지 못하고 줄을 서서 기다리게 됩니다. count가 다시 +1이 되면 기다리는 스레드가 순서대로 접근이 허용됩니다.

```swift
let semaphore = DispatchSemaphore(value: 1) // count = 1

DispatchQueue.global().async {
    semaphore.wait() // count -= 1

    semaphore.signal() // count += 1
}
```

- wait()는 값에 접근했다고 알리는 메서드
- signal()은 볼 일 다봤다는 메서드

DispatchSemaphore를 사용할 때에는 주의 사항이 있습니다. 반드시 wait()와 signal()을 짝지어서 호출해주어야 합니다.

```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let semaphore = DispatchSemaphore(value: 1)

DispatchQueue.global().async {
    for _ in 1...3 {
        semaphore.wait()
        let card = cards.removeFirst()
        print("야곰: \(card) 카드를 뽑았습니다!")
        semaphore.signal()
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        semaphore.wait()
        let card = cards.removeFirst()
        print("노루: \(card) 카드를 뽑았습니다!")
        semaphore.signal()
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        semaphore.wait()
        let card = cards.removeFirst()
        print("오동나무: \(card) 카드를 뽑았습니다!")
        semaphore.signal()
    }
}
```

semaphore의 value를 1로 설정하여 하나의 스레드만 접근이 가능하도록 초기화해주었습니다. 그리고 각 스레드에서 cards에 접근하기 전과 후에 semaphore의 wait()과 signal()을 호출해주어 다른 스레드의 접근을 막아주고 있는 것이지요. 이때 value를 2로 설정하게 된다면 2개의 스레드에서 접근이 가능합니다. 

DispatchSemaphore는 스레드의 접근을 제어하기 위한 수단이며 꼭 Race Condtion을 해결하기 위한 수단만은 아니라는 점!!!

### Serial Queue 활용하여 race condition 해결하기
DispatchSemaphore를 활용하여 Race Condtion을 해결할 수도 있지만, 다른 방법으로 Race Condition을 해결해볼 수도 있습니다.

Race Condition이 발생하는 이유는 여러 스레드에서 질서없이 배열에 접근했기 때문이었습니다.
Serial Queue를 사용하게 되면 하나의 스레드에서만 작업을 처리하기 때문에 질서가 생기게 되는 것입니다.

우리는 Serial Queue를 만들고 각 작업들을 Serial Queue에 넣어주면 됩니다. 
```swift
import Foundation

var cards = [1, 2, 3, 4, 5, 6, 7, 8, 9]
let pickCardsSerialQueue = DispatchQueue(label: "PickCardsQueue")

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("야곰: \(card) 카드를 뽑았습니다!")
        }
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("노루: \(card) 카드를 뽑았습니다!")
        }
    }
}

DispatchQueue.global().async {
    for _ in 1...3 {
        pickCardsSerialQueue.sync {
            let card = cards.removeFirst()
            print("오동나무: \(card) 카드를 뽑았습니다!")
        }
    }
}
```

## UI 작업은 왜 Main Thread에서 해야할까요?
UI를 그리는 등 UI와 관련된 작업은 main 스레드에서 해야한다고 했습니다. 심지어 UI 작업을 main 스레드에 할당하지 않으면 에러가 일어나곤 합니다. UIKit 공식문서에서도 아래와 같은 내용의 문구가 있습니다.

> Important
Use UIKit classes only from your app’s main thread or main dispatch queue, unless otherwise indicated. This restriction particularly applies to classes derived from UIResponder or that involve manipulating your app’s user interface in any way. UIKit 공식문서

>해석: UIKit 클래스는 main thread에서만 사용하세요. 이러한 제한 사항은 UIResponder에서 파생된 클래스 혹은 사용자가 UI를 통해 조작하는 것과 관련된 클래스에 적용되는 사항입니다.
도대체 어떤 이유에서 이러한 제한을 두는 것일까요?

1️⃣ UIKit은 Thread Safe하지 않습니다.
UIKit은 UI를 구성하는 구성 요소들의 집합입니다. 즉 화면에 즉각적으로 표현이 되는 요소들이지요. 그러나 이 UIKit의 대부분의 요소들은 Thread Unsafe한 성질을 가지고 있습니다. 여러 스레드에서 접근이 가능한 것이지요. 

앞에서 배웠던 것처럼 Race Condition이 발생하게 됩니다. 우리가 보는 화면의 상태를 여러 스레드에서 접근하여 제어하는 것입니다. 버튼의 모양을 바꾸는 일과 버튼의 위치를 이동시키는 일, 버튼의 색을 바꾸는 일 등이 충돌하여 우리의 의도대로 동작하지 않을 수 있습니다. 그러므로 우리와 같은 개발자들은 Thread Unsafe한 UIKit의 작업을 Serial Queue인 Main Thread로 가져와서 작업을 해야합니다.

2️⃣ 그렇다면 꼭 Main Thread여야하는 이유가 있나요? 다른 직렬 큐에 넣으면 안되나요?
UI작업을 꼭 메인 스레드에서 작업해야하는 이유는 메인 스레드에는 Main RunLoop가 동작하고 있기 때문입니다. 메인 스레드에서는 RunLoop가 일정한 주기를 유지하며 계속 동작하고 있습니다. 이 주기에 맞추어서 사용자의 입력을 받아서 UI를 그리게 됩니다. 이러한 주기를 View Drawing Cycle이라고 합니다. 하지만 사실 모든 스레드는 각자의 RunLoop를 가질 수 있는데요, 그 이유는 모든 스레드의 RunLoop에 따라 각자가 UI를 그리게 된다면 UI가 그려지는 시점이 모두 제각각이 되기 때문입니다. 그렇게 되면 비효율적일 뿐더러 위에서 언급한대로 Race Condition이 발생하게 되지요. 따라서 기준을 하나로, Main RunLoop의 기준에 따르는 것으로 문제를 해결할 수 있습니다.

## 참고
> 관련 주소 : https://yagom.net/courses/동시성-프로그래밍-concurrency-programming/lessons/gcd-심화/