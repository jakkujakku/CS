# ✅ 목차
0. GCD
1. DispatchQueue 1 : Serial / Concurrent
2. Main Thread
3. DispatchQueue 2 : sync / async
4. DispatchQueue 3 : 그 외의 다양한 기능

## GCD 
- Grand Central Dispatch의 약자로 멀티 코어 환경과 멀티 스레드 환경에서 최적화된 프로그래밍을 할 수 있도록 애플이 개발한 기술. 
- 개발자가 직접 코어와 스레드를 관리하지 않아도 시스템에서 알아서 관리를 해주는 것

## DispatchQueue 1 : Serial / Concurrent
```swift
class DispatchQueue : DispatchObject

- Disaptch: 보내다(파견하다) 
- Queue: 대기열
```

- DispatchQueue는 대기열에 보내다라는 뜻 
- DispatchQueue는 GCD를 사용하기 위한 대기열로, GCD 기술의 일부입니다. 우리는 이 대기열 들에 작업을 추가해주기만 하면 시스템은 알아서 스레드를 관리하여 작업을 처리하도록 도와줄 것입니다.  
- DispatchQueue는 FIFO(First In, First Out)로 작업을 처리합니다.
- DispatchQueue에 작업을 넘길 때 2가지를 꼭 정해주어야 합니다. -> 다중/단일 , 동기/비동기 여부
    - 단일 스레드를 사용할 것인가, 
    - 다중 스레드를 사용할 것인가(Serial/Concurrent) 
    - 동기로 작업을 처리할 것인가, 
    - 비동기로 작업을 처리할 것인가(sync/async) 

> DispatchQueue가 GCD와 동치되는 것이라 생각할 수도 있는데, 사실 GCD는 더 넓은 개념임. GCD는 Dispatch라는 프레임워크와 동치시킬 수 있음. DispatchQueue는 이 중의 일부일 뿐임.


### Serial / Concurrent
DispatchQueue는 크게 Serial Queue와 Concurrent Queue로 두 가지로 구분할 수 있음
Serial은 단일 스레드에서만 작업을 처리하고, 
Concurrent는 다중 스레드에서 작업을 처리합니다.

DispatchQueue를 초기화할 때 attributes를 따로 .concurrent로 설정하지 않으면 그 기본 값은 Serial이 됩니다

파라미터 label을 받는 초기화 구문들은 모두 커스텀 DispatchQueue의 초기화 메서드입니다. 

```swift
// Serial Queue
DispatchQueue(label: "Serial")
DispatchQueue.main
// main은 전역적으로 사용되는 Serial DispatchQueue 입니다.

// Concurrent Queue
DispatchQueue(label: "Concurrent", attributes: .concurrent)
DispatchQueue.global()
```

> 그런데 main은 프로퍼티, global()은 메서드?
- Main Thread는 다음페이지에서 한 번 더 다루겠지만, 메모리에 늘 올라와 있는 Default 스레드입니다. 하지만 여러 스레드에서 여러 작업을 동시에 하는 Concurrent Queue는 Main Thread 외에 새로운 스레드를 만들어서 사용하게 됩니다. 

### main / global
DispatchQueue.main과 DispatchQueue.global()은 이미 만들어져있는 큐로 각각 Serial, Concurrent 큐입니다. 

특히 main은 일반적인 Serial 큐와는 달리 앱이 실행되는 동안에는 늘 메모리에 올라와 있으며 또 전역적으로 사용 가능한 큐라는 특수한 성질이 있습니다. 

main에 작업을 추가하면 Serial 큐인 main 스레드에서 작업을 처리하게 됩니다. 
하나의 스레드에 작업이 쌓여서 처리가 되는 것이지요. 
따라서 main 스레드에만 작업을 쌓을 경우 동시에 여러 작업을 처리할 수 없습니다. 
Serial Queue이기 때문입니다. 
반면 global에 작업을 추가하면 새로운 스레드를 만들어 그 위에서 작업을 처리합니다. 
Concurrent Queue이기 때문입니다.

global 스레드는 main 스레드가 아닌, 작업을 처리하기 위해 발생한 스레드들을 말합니다. 
global 스레드는 main 스레드와는 달리 global()이 호출되면 작업을 처리하기 위해 메모리에 올라왔다가, 작업이 끝나고 나면 메모리에서 제거됩니다. 

DispatchQueue로 GCD를 구현하기 위해서는 2가지를 정해주어야한다고 했습니다. 이렇게 Serial Queue에서 작업할 건지, Concurrent Queue에서 작업할 것인지를 정해주었다면 다음으로는 동기/비동기 처리를 정해주면 됩니다. 

```swift
// 동기, sync
DispatchQueue.main.sync {}
DispatchQueue.global().sync {}

// 비동기, async
DispatchQueue.main.async {}
DispatchQueue.global().async {}
```

## Main Thread
메인 스레드는 앱의 기본이 되는 스레드입니다. 모든 작업은 스레드 위에서 처리되기 때문입니다. 
메인 스레드는 앱의 생명주기와 같은 생명주기를 가지는, 앱이 실행되는 동안에는 늘 메모리에 올라와있는 기본 스레드입니다. 동시성 프로그래밍에서는 여러 개의 스레드를 사용한다고 했습니다. 이 경우에도 메인 스레드는 늘 메모리에 올라온 상태로 존재하며 메인 스레드에서부터 필요한 만큼의 스레드가 파생되는 것입니다. 이때 파생되는 스레드들은 자신이 담당하는 작업이 처리되면 메모리에서 사라지게 됩니다. 

### 메인 스레드의 몇 가지 특징 
- 전역적으로 사용이 가능합니다.
- global 스레드들과는 다르게 Run Loop가 자동으로 설정되고 실행됩니다. 
- 메인 스레드에서 동작하는 Run Loop를 Main Run Loop라고 합니다.

## DispatchQueue 2 : sync / async
```swift
// 동기, sync
DispatchQueue.main.sync {}
DispatchQueue.global().sync {}

// 비동기, async
DispatchQueue.main.async {}
DispatchQueue.global().async {}
```

- 동기는 작업이 처리되기를 기다리는 것
- 비동기는 작업이 처리되기를 기다리지 않고 다른 스레드에서 처리되는 것

sync는 동기로 작업을 처리하겠다는 메서드 
뒤에 오는 코드 블록이 하나의 작업이 됩니다. 

예를 들어 DispatchQueue.global().sync {}를 호출한다는 것은 main 스레드 말고 새로운 스레드를 만들어서 작업을 처리할거야. 그런데 내 작업이 끝날 때까지 기다려야해!라는 의미가 됩니다.

async는 비동기로 작업을 처리하겠다는 메서드입니다. 
작업이 끝나기를 기다리지 않는다는 이야기입니다. 

예를 들어 DispatchQueue.global().async {}를 호출한다는 것은 main 스레드 말고 새로운 스레드를 만들어서 작업을 처리할건데, 다음 작업이 있다면 날 기다리지 말고 처리해도 좋아. 나는 새 스레드에서 알아서 작업할게!라는 의미가 됩니다.

```swift
// main.async
import Foundation

DispatchQueue.main.async {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.main.async {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

/* - 출력
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
*/

```

main 스레드에 비동기(async)로 일을 처리하는 코드입니다. 다른 스레드를 새로 만들지 않고 main 스레드에서만 일을 처리하고 있지요.  
작업이 끝나기를 기다리지는 않고 있지만 main 스레드라는, 단일 스레드에서만 작업이 이루어지고 있기 때문에 동시에 작업이 처리되지는 못하는 것입니다

```swift
import Foundation

DispatchQueue.main.async {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

for _ in 1...5 {
    print("🥶🥶🥶🥶🥶")
    sleep(2)
}

// 출력 - 어떤 것이 먼저 출력될지 모름..
```

단일 스레드 환경이지만 비동기(async)로 코드 블록을 호출했기 때문에 작업이 끝나기를 기다리지 않고 다음 코드로 넘어가버림.

이때 다음 코드가 먼저 실행이 된다면 😀😀😀😀😀의 출력은 그 다음으로 미루어짐. 왜냐하면, 단일 스레드에서 처리되는 것이기 때문임

```swift
// global().async

import Foundation

DispatchQueue.global().async {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.global().async {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

DispatchQueue.main.async {
    for _ in 1...5 {
        print("🥵🥵🥵🥵🥵")
        sleep(1)
    }
}

/* - 출력 (랜덤)
😀😀😀😀😀
🥶🥶🥶🥶🥶
🥵🥵🥵🥵🥵
😀😀😀😀😀
🥵🥵🥵🥵🥵
😀😀😀😀😀
🥶🥶🥶🥶🥶
🥵🥵🥵🥵🥵
😀😀😀😀😀
🥵🥵🥵🥵🥵
🥶🥶🥶🥶🥶
😀😀😀😀😀
🥵🥵🥵🥵🥵
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
*/
```

main 스레드가 아닌 다른 스레드를 만들어 비동기적으로 작업을 처리해주고 있습니다. 출력 결과를 보면 우리가 그토록 학습했던 동시에 작업을 처리하는 코드입니다.

하지만 이때 어떤 코드가 먼저 실행될지는 예측할 수가 없습니다. 바로 위에서 언급했던 async의 특성 때문이지요. 또한 각 스레드마다 작업이 처리되는 속도가 다를 수 있으며 이는 직접 통제할 수는 없습니다. 

global().async를 이미지로 표현하면 아래와 같습니다. global 스레드에서 작업이 끝나면 해당 스레드는 메모리에서 제거됩니다.

```swift
// global().sync

import Foundation

DispatchQueue.global().sync {
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}

DispatchQueue.global().sync {
    for _ in 1...5 {
        print("🥶🥶🥶🥶🥶")
        sleep(2)
    }
}

for _ in 1...5 {
    print("🥵🥵🥵🥵🥵")
    sleep(1)
}
// main 스레드에서 동작하는 코드

/* - 출력
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
😀😀😀😀😀
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥶🥶🥶🥶🥶
🥵🥵🥵🥵🥵
🥵🥵🥵🥵🥵
🥵🥵🥵🥵🥵
🥵🥵🥵🥵🥵
🥵🥵🥵🥵🥵
*/
```
global() 메서드는 새로운 스레드를 생성해주고, 그 위에서 작업을 처리해주는 메서드

위 코드는 main 스레드가 아닌 다른 스레드를 만들어 동기적으로 작업을 처리해주는 코드입니다. 

세 코드 블록의 스레드는 각각 다르지만 동기적으로 일을 처리하기 때문에 각각의 작업이 끝나기를 기다립니다. 

따라서 코드 블록이 작성된(호출된) 순서대로 이모지가 출력되고 있지요. 

### main.sync
main.sync는 제대로 동작하지 않는 코드라고 했습니다. 정확히는 main 스레드에서 직접 호출하면 안되는 코드입니다. 플레이 그라운드에서 main.sync를 호출해주면 아래와 같은 에러가 발생함.
```swift
DispatchQueue.main.sync { /* error: error: Execution was interrupted, reason: EXC_BREAKPOINT (code=1, subcode=0x18011922c).
    The process has been left at the point where it was interrupted, use "thread return -x" to return to the state before expression evaluation. */
    for _ in 1...5 {
        print("😀😀😀😀😀")
        sleep(1)
    }
}
```

main.sync를 직접적으로 호출하면 deadlock(교착상태)에 빠지게 됩니다. 이는 위에서 언급했던 작업이 끝나기를 기다리지 않는 async의 특성과는 반대되는, 작업이 끝나기를 기다리는 sync의 특성때문에 발생합니다. 

위에서 몇 가지 코드를 작성하면서 sync는 코드 블록이 처리되기 전까지 다음 코드로 넘어가지 않는 것을 확인했습니다. 이러한 상황을 Block-wait이라고 하는데요, 코드 블록이 끝나기 전까지 그 스레드는 멈춰있겠다는 뜻입니다. 

따라서 main 스레드에서 main.sync를 호출하게 되면 main 스레드는 sync의 코드 블록이 수행되기를 기다려야 합니다. 하지만 이때 sync의 코드 블록 역시 멈춰버리는 것이지요. 

main 스레드에서 실행되고 있던 코드이기 때문입니다. 따라서 아무것도 실행되지 못하고 main 스레드는 sync가 끝나기를, sync는 main 스레드의 Block-wait이 끝나기를 기다리는 상태가 되어버립니다. 

이러한 현상은 main 큐이기 때문에 발생하는 현상입니다. 만약 Serial 큐를 커스텀하여 sync를 실행한다면 에러는 발생하지 않지요. 이때는 main 스레드와는 별개의 문제가 되기 때문입니다.

하지만 main.sync를 호출할 수 있는 경우도 있습니다. main 스레드에서 호출하지 않으면 됩니다. 

예를 들어 global 스레드에서 main.sync를 호출한다면 에러없이 예상대로 동작하는 것을 확인할 수 있습니다.

```swift
import Foundation

DispatchQueue.global().async {
    DispatchQueue.main.sync {
        for _ in 1...5 {
            print("😀😀😀😀😀")
            sleep(1)
        }
    }
}

for _ in 1...5 {
    print("🥶🥶🥶🥶🥶")
    sleep(2)
}
```

## DispatchQueue 3: 그 외 다양한 기능들
### DispatchWorkItem
DispatchQueue에서 코드 블록을 호출하는데에 있어서 DispatchWorkItem을 활용하여 코드 블록을 캡슐화해줄 수 있습니다.  

하지만 DispatchWorkItem을 이용하면 타입을 명시하는 동시에 더 직관적인 코드를 작성할 수 있습니다. 정의된 DispatchWorkItem는 sync와 async 메서드의 execute 파라미터를 통해 전달하면 됩니다

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

DispatchQueue.main.async(execute: yellow)
DispatchQueue.global().sync(excute: blue)
.

그럼 main/global, sync/async를 복습해볼 겸 아래의 코드들은 어떻게 동작할지 유추해보고 직접 실행해봅시다!

// 1
DispatchQueue.global().async(execute: yellow)
DispatchQueue.global().sync(execute: blue)
DispatchQueue.main.async(execute: red)
// 2
DispatchQueue.global().sync(execute: yellow)
DispatchQueue.global().async(execute: blue)
DispatchQueue.main.async(execute: red)
// 3
DispatchQueue.main.async(execute: yellow)
DispatchQueue.global().async(execute: blue)
DispatchQueue.global().sync(execute: red)
// 4
DispatchQueue.main.async(execute: yellow)
DispatchQueue.global().sync(execute: blue)
DispatchQueue.global().async(execute: red)
```

### asyncAfter
- asyncAfter은 async 메서드를 원하는 시간에 호출해줄 수 있는 메서드

```swift
DispatchQueue.global().asyncAfter(deadline: .now() + 5, execute: yellow)

이 코드는 지금(.now())으로부터 5초 후에 yellow라는 DispatchWorkItem을 실행시킨다는 코드입니다. execute 파라미터 대신 직접 코드 블록을 구현해도 됩니다. 

deadline 대신 wallDeadline이라는 파라미터를 사용해줄 수 있는데, wallDeadline은 시스템(기기)의 시간을 기준으로 카운트를 하는 것입니다. 

즉, deadline은 스톱워치로 측정하듯이 5초를 카운트해서 작업이 시작되고, wallDeadline은 지금 5시니까 5시 5초에 작업을 시작해야지 와 같이 작업을 수행하는 것입니다. 

DispatchQueue.global().asyncAfter(deadline: .now() + 5, execute: yellow)
DispatchQueue.global().asyncAfter(wallDeadline: .now() + 5, excute: blue)
```

### asyncAndWait
- asyncAndWait 메서드를 사용하면 비동기 작업이 끝나는 시점을 기다릴 수 있습니다. 비동기로 처리되는 어떤 동작이 끝나기를 의도적으로 기다려야할 때 사용할 수 있습니다. 동작하는 논리는 사실 sync와 많이 유사합니다.
```swift
DispatchQueue.global().asyncAndWait(execute: yellow)
print("Finished!")
```

## 참고
- 참고 주소 - 1 : https://yagom.net/courses/동시성-프로그래밍-concurrency-programming/lessons/gcd-기본/
- 참고 주소 - 2 : https://sujinnaljin.medium.com/ios-차근차근-시작하는-gcd-grand-dispatch-queue-1-397db16d0305