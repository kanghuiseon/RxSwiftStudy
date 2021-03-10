# Chapter 2 - Observables
## What is an observable?
* observables는 Rx에서 가장 중요한 요소이다. 
* Rx에서 'observable'은 'observable sequence', 'sequence'로 바꿔서 사용할 수 있다. 이것들은 ***비동기적*** 이다.
* observables는 일정 기간동안에 ***이벤트*** 를 생성하는데, 이것을 ***방출(emission)*** 이라고도 한다.
* 이벤트들은 값을 가질 수 있다. 예를 들어, 숫자나 custom type의 instance들, 또는 제스처나 탭도 값이 될 수 있다.
* observables를 가장 잘 표현하는 방법 중 하나는 marble diagrams이다. 이것은 timeline위에서 값이 어떤 순서로 진행이 되는지를 그림으로 보여준다. 
<img src = "1" height = 50>

* 화살표는 시간을 나타내고, 동그라미는 하나의 이벤트, 그안의 숫자는 값을 나타낸다.
* 1-2-3 순서대로 방출된다.
* [[참고] RxMarble](https://rxmarbles.com)


## Lifecycle of an observable
* 위의 다이어그램에서는 세 개의 요소들을 방출한다. 
* observable은 ***next*** 이벤트를 통해 하나의 요소를 방출한다.
<img src = "2" height = 50>

* 또 다른 marble diagram을 보면, 가장 오른쪽에 수직바를 볼 수 있는데, 이것은 observable이 정상적으로 종료되었다는 것을 말한다.
* observable은 세 개의 tap 이벤트를 방출하고나서 끝이난다. 이때 ***completed*** 이벤트가 발생하고 observable이 종료된다.
* 종료되고나서는 어떠한 이벤트도 방출하지 않는다. 

<img src = "3" height = 50>

* 하지만 정상적으로 종료되지 않을 때도 있다. 이때에는 위에서 보는것처럼 error를 포함한 error 이벤트를 방출한다.
* 만약 에러를 방출한다면, 이때에도 동일하게 observable은 종료되고 더 이상 어떤 것도 방출할 수 없다.

>>> 결론 
>>> * observable은 elements를 가진 ***next*** 이벤트를 방출한다.
>>> * 이벤트가 종료될때까지 (***error*** or ***completed*** event가 발생할때까지) 계속해서 next 이벤트를 방출한다.
>>> * observable이 종료되면, 더 이상 이벤트를 방출하지 않는다.

```swift
/// Represents a sequence event.
///
/// SEquence grammar:
/// **next\* (error | completed)**
public enum Event<Element> {
    /// Next element is produced
    case next(Element)
    
    ///Sequence terminated with an error.
    case error(Swift.Error)
    
    /// Sequence completed successfully.
    case completed
}
```
* 이벤트들은 enum case로 정의되어있다. 


## Creating observables
* 다음은 observable을 생성하는 연산자이다.
1. just
```swift
let one = 1
let two = 2
let observable = Observable<Int>.just(one)
let observable1 = Observable.just([one, two])
```
-> 만약 한가지 요소만을 방출하는 observable을 생성하고 싶다면 just를 사용한다.
-> 여기서 just는 observable의 타입 메소드로, 파라미터로 받은 하나의 요소 그대로 방출한다.
-> just를 메소드라고 부를 수도 있지만, Rx에서는 메소드 대신 operators(연산자)라고 부른다.
-> just에서 배열을 파라미터로 넘겨줄 수도 있는데, 이때에는, 배열 내부에 값이 여러개더라도 파라미터로는 ***배열 하나*** 만 넘겨주기 때문에 가능한 것이다. 

2. of
```swift
let observable2 = Observable.of(one, two, three)
let observable3 = Observable.of([one, two, three])
```
-> just가 하나의 요소를 방출하는 연산자라면, of는 하나 이상의 연산자를 방출하는 연산자이다.
-> 또한 type을 명확하게 지정해주지 않아도, 파라미터로 추론이 가능하다.
-> 만약 array를 전달하고 싶다면, observable3와 같이 파라미터로 배열을 넘겨주면 된다.

3. from 
```swift
let observable4 = Observable.from([one, two, three])
```
-> from 연산자는 파라미터로 넘겨진 array내부의 값들을 가진 Observable을 return한다.
-> from 연산자는 오직 배열만 파라미터로 받는다.



## Subscribing to observables
* iOS 개발자라면 NotificationCenter에 익숙할텐데, 이것은 어떤 이벤트가 발생했을때 Observer에게 notification을 보낸다. 잠깐 NotificationCenter의 예시를 살펴보자면,
```swift
let observer = NotificationCenter.default.addObserver(
  forName: UIResponder.keyboardDidChangeFrameNotification,
  object: nil,
  queue: nil) { notification in
  // Handle receiving notification
}
```
-> keyboardDidChangeFrameNotification의 옵저버를 생성하고, trailing closure로 전달된 handler를  수행하도록 한다.

* RxSwift에서의 subscribing도 이와 유사하다. 대신 observable을 구독하고 싶을땐, addObserver가 아닌, subscribe()를 이용한다.
* 또한 .default singleton 인스턴스를 사용하는 NotificationCenter와 달리, Rx에서는 여러개의 observerble가 존재할 수 있다.

* ***observable은 events를 보내지 않고, 이벤트의 순서만을 결정할 뿐이다. 어떤 동작도 하지 않고, observer가 observerble을 구독한 순간에만 이벤트를 전달한다.***


### subscribe
```swift
let sequence = 0..<3
var iterator = sequence.makeIterator()

while let n = iterator.next(){
    print(n)
}

/* prints:
0
1
2
*/
```
-> observable을 구독하는 것은 Swift standard library의 Iterator.next()와 유사하다. next와 같이 순서대로 이벤트를 방출한다.

* 또한, 각 event type에 따라 handler를 추가할 수도 있다. 
* observable은 next, error, completed 이벤트를 방출하는데, next 이벤트는 요소를 handler에게 전달할 수 있고, error이벤트는 error instance를 포함한다. 
* 이것을 코드로 설명해보자면,
```swift
let observable = Observable.of(1,2,3)
observable.subscribe{ event in 
    print(event)
}

/* prints:
next(1)
next(2)
next(3)
completed
/*
```
-> 위의 코드는 prints대로 출력된다. observable은 of(1,2,3)의 각 요소에 대한 next 이벤트를 방출하고, 마지막으로 completed 이벤트를 방출한 후, 종료된다.
```swift
observable.subscribe { event in
    if let element = event.element{
        print(element)
    }
}
/* prints:
1
2
3
/*
```
-> 만약 이벤트가 아닌 element를 출력하고 싶다면, 위의 코드와 같이 if let 구문을 이용하여 element에 접근한다.
-> 이 pattern은 자주 사용되기 때문에 RxSwift에서는 아래의 코드와 같이 if let을 사용하지 않는 방법을 제시한다.
```swift
observable.subscribe(onNext: { element in 
    print(element)
})
```
-> subscribe 연산자는 next, error, completed 각각에 대해 다룰 수 있지만, 위의 코드는 next event만을 취하고, 다른 이벤트들은 모두 무시한다.
-> onNext 클로저는 next 이벤트의 element를 argument로 받아서, 처리한다.


### empty()
```swift 
let observable = Observable<Void>.empty()
    .subscribe{ print($0) }
  
/* prints:
completed
/*  
```
-> empty연산자는 어떠한 요소도 방출하지 않고, completed 이벤트를 전달하는 observable을 생성한다.
-> 여기서 observable은 명확한 타입으로 정의가 되어야 한다. 왜냐하면 empty 연산자는 타입을 추론할 요소가 없기 때문이다. (보통  Void로 많이 쓰임)
-> 바로 종료되는 observable을 return 하기를 원하거나, 0개의 값을 가지는 observable을 return 하기 원할때 쓰인다.



### never()
```swift
let observable = Observerble<Void>.never()
observable.subscribe(
    onNext: { element in
        print(element)
    },
    onCompleted: {
        print("Completed")
    }
)
```
-> empty와는 반대로 never 연산자는 아무것도 방출하지 않고, 절대로 종료되지 않는 observable을 생성한다.
-> infinite duration을 나타낼때 사용된다.  
-> 아무것도 출력되지 않는다.

### range()
```swift
let observable = Observable<Int>.range(start: 1, count: 10)
observable
    .subscribe(onNext: { i in  
      // 2
      let n = Double(i)
      
      let fibonacci = Int(
        ((pow(1.61803, n) - pow(0.61803, n)) /
          2.23606).rounded()
      )
      print(fibonacci)
})
```
-> 만약 시작값에서부터 1씩 증가하는 sequence를 방출하는 observable을 생성하고 싶다면, range 연산자를 사용한다.
-> start에는 시작값, count에는 시작값에서부터 1씩 몇번 증가할 것인지를 넘겨준다.



## Disposing and terminating
>>> 중요한 사실
>>> observable은 어떠한 구독자가 생기기 전까지는 어떠한 액션도 취하지 않는다. 한 observer가 구독을 시작하자마자 observable은 completed 또는 error이벤트가 발생할때까지 next이벤트를 방출한다. 

### dispose()
* 하지만 completed나 error이벤트가 아니라, 구독을 취소함으로써도 observable을 중지시킬수가 있다. 아래의 코드에서,
```swift
    // 1
  let observable = Observable.of("A", "B", "C")
  
    // 2
  let subscription = observable.subscribe { event in
        // 3
        print(event)
  }
```
-> string 타입의 observable을 생성한다.
-> subscribe 연산자는 Disposable을 리턴하고 이것을 상수인 subscription에 저장한다.
-> 또한 observable이 이벤트를 방출할때마다, handler에서는 각 이벤트를 출력하도록 한다.
-> 여기서, 명확하게 구독을 취소하고 싶다면, ***dispose()*** 를 호출한다. 호출 이후에는 observable은 더 이상 이벤트를 방출하지 않는다.
```swift
subscription.dispose()
```

### DisposeBag

* dispose()를 이용해서, 구독을 전부 관리하는 것은 귀찮은 일이므로, RxSwift는 DisposeBag 타입을 제공한다.
* dispose bag은 disposables를 가지고 있는데, 보통 disposed(by:) 메소드를 이용해서 추가된다. 아래의 예시를 살펴보자
```swift
let disposeBag = DisposeBag()
Observable.of("A", "B", "C")
    .subscribe {
        print($0)
    }
    .disposed(by: disposeBag)
}
```
-> 우선 DisposeBag 인스턴스와 observable을 생성한다.
-> observable을 구독하여, 이벤트를 출력하도록 한다.
-> 마지막으로, subscribe에서 return 된 Disposable을 disposeBag에 추가해준다.

* 이 방식은 매우 자주 쓰인다. 그런데 왜 이런 방식을 사용해야하는건가?
-> 그 이유는 메모리 문제이다. 만약 dispose()를 호출하지 않거나, disposeBag에 추가하지 않는다면 사용했던 리소스가 해제되지 않을 것으고, 이 리소스들은 메모리를 차지하고 있게 된다. 
-> 하지만 swift compiler는 똑똑하기 때문에 사용하지 않는 Disposables에 대해서는 경고를 줄것이다.


### Create
<img src = "4" height = 50>
-> create 연산자는 observable이 방출할 모든 이벤트를 명시하는 방법이다.
-> create 연산자는 subscribe라는 이름의 파라미터를 받는데, 이것은 직접 방출할 이벤트들을 직접 구현한다
-> subscribe는 AnyObject를 파라미터로 받고, Disposable을 리턴하는 escaping closure이다. AnyObserver는 구독자들에게 방출될 observable sequence에 값을 추가하는 generic type이다. 

```swift
Observable<String>.create { observer in
      // 1
      observer.onNext("1")
      
      // 2
      observer.onCompleted()
      
      // 3
      observer.onNext("?")
      
      // 4
      return Disposables.create()
}
```
-> 1. observer에 next 이벤트를 추가한다. onNext(_:)는 on(.nex-t(_:)) 의 편리한 방법이다.
-> 2. 또한 completed 이벤트를 추가한다. 이것도 on(.completed)의 편리한 방법이다.
-> 3. 또 다른 nex이벤트를 추가한다.
-> 4. disposable을 return 한다.
-> 마지막 4번에서는 subscribe 연산자는 구독을 의미하는 disposable을 return 해야 하기 때문에 Disposables.create()를 사용하여, disposable을 생성하여 리턴한다.

* 3번의 onNext 이벤트는 구독자에게 방출이 될까? 
* 답은 No!, 그 이유는 바로 위에서 onCompleted메소드를 호출하여 이미 해당 observable을 종료시켰기 때문에 더 이상의 방출은 발생하지 않는다.



## Creating observable factories
* 구독을 기다리는 observable을 생성하는것 외에 새로운 observable을 각 구독자에게 제공하는 observable factories를 만들 수도 있다.

### diferred
```swift
let disposeBag = DisposeBag()
// 1
var flip = false

// 2
let factory: Observable<Int> = Observable.deferred {
    // 3
    flip.toggle()

    // 4
    if flip {
        return Observable.of(1, 2, 3)
    } else {
        return Observable.of(4, 5, 6)
    }
}

//5
for _ in 0...3 {
      factory.subscribe(onNext: {
        print($0, terminator: "")
      })
      .disposed(by: disposeBag)

      print()
}
```
-> 위의 코드는 flip이라는 flag를 생성하고, 각 flag에 따라 다른 Observable을 return한다.
-> deferred 연산자는 특정 조건에 따라서 observable을 생성한다.
-> 5번 코드에서 factory를 4번 구독한 결과는 123\n456\n123\n456이다. 결과에서 보는 것과 같이 구독할때마다 flag가 토글되고 각 flag조건에 따라 다른 결과값을 출력하는 것을 볼 수 있다.


## Challenges
## 1. do
* observable이 방출하는 이벤트에 어떤 영향도 주지 않는 이 외의 작업을 하고 싶다면, ***do*** 연산자를 활용한다.
* do 연산자는 onSubscribe 핸들러를 가지고 있어서, 이곳에서 다른 작업을 수행하면 된다.
* 또한 do(onNext:onError:onCompleted:onSubscribe:onDispose)를 이용하여, 각 이벤트에 맞는 작업도 수행할 수 있다.

## 2. debug
* debug 연산자는 observable에 대한 모든 이벤트 정보를 출력한다. 
* 파라미터 값으로 identifier string 을 넘겨주어서 디버깅을 진행할 수 있다.
<img src= "5" height = 5>
