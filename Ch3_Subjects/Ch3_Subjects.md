# Ch.3 Subjects

Ch2에서 array와 비슷해보이는 Observable에 대해 배웠다 옵저버블을 어떻게 만들고 어떻게 구독하며 어떻게 디스포즈하는지 배웠지만, 옵저버블은 기본적으로 read-only이다 값을 추가할수가 없다. 

앱을 만들때 필요한건 실시간으로 새값을 옵저버블에 추가하고 구독자에게 방출하는 그런것이다. 그래서 등장하는 새개념이 관찰가능한 동시에 관찰자역할을하는, 옵저버블의 성질을 그대로 가지고 있으면서 새값을 추가할수 있는, subject이다.

# Getting Started: 시작하기

다음코드를 통해 간단한 subject의 동작을 확인할수 있다.

```swift
example(of: "PublishSubject") {

 	// 1
     let subject = PublishSubject<String>()

     // 2
     subject.on.(next("Is anyone listening?"))

     // 3
     let subscriptionOne = subject
         .subscribe(onNext: { (string) in
             print(string)
         })

     // 4
     subject.on(.next("1"))		
		 --- Example of: PublishSubject ---
		 1

     // 5
     subject.onNext("2")		
     2
 }

// print
--- Example of: PublishSubject ---
1
2
```

publishSubject를 생성해 값을 전달하고 구독을 통해 그 값을 print해보는 예제이다.

- publishSubject는 publish라는 이름처럼(뉴스 출판사를 떠올리면 좋다.) 정보를 받아서 구독자에게 바로 전달해주는 역할을 한다.
- observable과의 차이는 값을 추가할수 있는점이다. .on.next메서드를 통해 값을 추가한다.
- .on.next는 onNext로 줄여쓸수 있다.
- "Is anyone listening?"때는 구독자가 없기때문에 출력이 이루어 지지 않는다.
- 값의 출력은 (구독자에게 정보를 전달해주는시점은) 이벤트가 섭젝트에 추가되었을때 구독자에게 현재 이벤트를 전달해주기 때문에 구독시점자체에는 출력이 일어나지 않는다.

# What are Subjects? 서브젝트란?

위 예제에서 살펴봤듯 서브젝트는 관찰가능한동시에 관찰자로서 존재한다. 구독가능한 동시에 값이 추가가능하단 소리이다. 서브젝트는 next이벤트를 받고 구독자에게 해당이벤트를 방출한다.

RxSwift에는 4가지 종류의 서브젝트가 있다.

- PublishSubject: 초기값이 없이 시작하며 새 엘리먼트를 구독자에게 방출한다.
- BehaviorSubject: 초기값이 존재하며 구독자에게 초기값을 다시 replay해주거나 마지막 엘리먼트를 새 구독자에게 방출한다.
- ReplaySubject: 버퍼사이즈로 초기화되며 사이즈만큼 엘리먼트들이 유지된다. 새구독자에게 현재 버퍼의 엘리먼트들을 새구독자에게 방출한다.
- AsynchSubject: 오직 마지막 엘리먼트에 대해서만 방출이 이루어지며 해당 섭젝트가 completed될때만 방출이 이루어진다. 거의 쓰이지 않으며 이책에서는 다루지 않는다.

이외에도 두가지 타입의 relay란게 존재하며 relay는 이벤트들중 next 이벤트만 받아들이고 relay하는 서브젝트의 래핑 타입이라고 생각하면 좋다. 

- PublishRelay, BehaviorRelay 두타입이 존재한다.
- error, completed를 relay에 새값으로 추가할수 없기 때문에 종료되지 않는 무한한 작업에 적합하다.

# PublishSubject

PublishSubject는 구독된 순간부터 새로운 엘리먼트를 받고 싶을 때 적합하며 .completed나 .error 이벤트로 서브젝트가 종료될때까지 지속된다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/1.%20publishsubject.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/1.%20publishsubject.png?raw=true)

- 첫줄이 서브젝트이다.
- 두번째줄, 세번쨰줄의 subscriber들이 구독된 시점(점선)이후부터 방출된 새로운 엘리먼트들을 받는것을 볼수있다.

```swift
example(of: "PublishSubject") {
     let subject = PublishSubject<String>()
     subject.onNext("Is anyone listening?")

     let subscriptionOne = subject
         .subscribe(onNext: { (string) in
             print(string)
         })
     subject.on(.next("1"))
		 --- Example of: PublishSubject ---
		 1
     subject.onNext("2")
     2

     // 1
     let subscriptionTwo = subject
         .subscribe({ (event) in
             print("2)", event.element ?? event)
         })

     // 2
     subject.onNext("3")
     3
     2) 3
     // 3
     subscriptionOne.dispose()
     subject.onNext("4")
     2) 4
     // 4
     subject.onCompleted()
     2) completed
     // 5
     subject.onNext("5")
		 // 무시
     // 6
     subscriptionTwo.dispose()

     let disposeBag = DisposeBag()

     // 7
     subject
         .subscribe {
             print("3)", $0.element ?? $0)
     }
         .disposed(by: disposeBag)
     3) completed

     subject.onNext("?")

 }
```

위코드는 publishSubject에 세개의 구독자를 추가해 publishSubject의 특징을 보여준다.

- 구독자1이 dispose를 통해 종료되면 구독자 2만 남아 구독을 계속 하게 된다.
- 다만 서브젝트자체가 onCompleted()메서드를 통해 종료되면 구독자 3을 새로 추가하더라도 종료된 서브젝트는 다시 시작되지 않고 complted 이벤트만 방출된다.
- 즉 서브젝트가 종료되면 그후의 구독자들은 구독시점에 종료된 서브젝트로부터 completed이벤트를 바로 재방출받게 된다.

### 정리

publish Subject는 한번 종료되면 미래의 구독자들에게 completed 이벤트를 재방출한다. 따라서 종료될때의 핸들러외에도 이미 종료된 서브젝트를 구독하는 경우에 대한 핸들러를 추가하는게 오류를 잡는데 도움이 된다.

사용예시로는 시간에 민간함 어플, 특정시간이후에 들어오는 접근을 막아야할때 사용할수 있을수 있다.

# ReplaySubject

퍼블리시서브젝트와 유사하나 구독시점에 가장마지막을 방출된 next 이벤트를 replay한다는 점에서 다르다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/2.%20behaviorsubject.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/2.%20behaviorsubject.png?raw=true)

```swift
// 1
enum MyError: Error {
    case anError
}

// 2
func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
    print(label, (event.element ?? event.error) ?? event)
}

//3
example(of: "BehaviorSubject") {

    // 4
    let subject = BehaviorSubject(value: "Initial value")
    let disposeBag = DisposeBag()
    // 6
    subject.onNext("X")

    // 5
    subject
        .subscribe{
            print(label: "1)", event: $0)
        }
        .disposed(by: disposeBag)
		--- Example of: BehaviorSubject ---
		1) X

    // 7
    subject.onError(MyError.anError)
		1) anError
    // 8
    subject
        .subscribe {
            print(label: "2)", event: $0)
        }
        .disposed(by: disposeBag)
    2) anError
}
```

- publishSubject와 다른점은 구독시점에 마지막 엘리먼트를 방출한다는 점외에는 다를게 없다. (마지막 엘리먼트 X가 방출된다.)
- error가 났을때 종료되는 방식은 퍼블리쉬서브젝트와 마찬가지로 이미 종료된 서브젝트에 대해서는 error이벤트를 재방출 받아 구독자2는 구독하자마자 error이벤트를 받는다.
- 또한 behaviorSubject는 선언시에 초기값을 가지며 onNext로 X를 주지 않았다면 구독자1의 구독시점에 마지막 엘리먼트인 "InitialValue"가 출력됐을것이다.

### 정리

UI를 구성할때 최신 정보를 가져와 UI를 업데이트 하기전에 사용하는 용도로 비헤이비어 서브젝트를 이용할수 있다. 

# ReplaySubject

replaySubject는 behaviorSubject의 확장판과 같다고 생각하면 좋다. 구독시점에 마지막엘리먼트뿐만이 아닌 버퍼사이즈로 정해준만큼의 엘리먼트를 옵저버블로부터 재방출 하도록 한다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/3.%20replaysubject.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/03_Subjects/3.%20replaysubject.png?raw=true)

- 하나 유념할점들은 이러한 버퍼들은 메모리를 가지고 있기때문에 엘리먼트로 이미지나 array같이 메모리를 크게 차지하는 값을 큰사이즈의 버퍼로 가지는것은 메모리에 큰 부하를 주므로 주의해야한다!

```swift
example(of: "ReplaySubject") {
  // 1
  let subject = ReplaySubject<String>.create(bufferSize: 2)
  let disposeBag = DisposeBag()
// 2
  subject.onNext("1")
  subject.onNext("2")
  subject.onNext("3")
// 3
  subject
    .subscribe {
      print(label: "1)", event: $0)
    }
    .disposed(by: disposeBag)
	--- Example of: ReplaySubject ---
	1) 2
	1) 3
	
	subject
    .subscribe {
      print(label: "2)", event: $0)
    }
    .disposed(by: disposeBag)
	2) 2
	2) 3

	subject.onNext("4")
	1) 4
	2) 4
	subject.onError(MyError.anError)
  1) anError
	2) anError
	
	subject
  .subscribe {
    print(label: "3)", event: $0)
  }
  .disposed(by: disposeBag)
	3) 3
  3) 4
  3) anError
}
```

코드를 살펴보면 behaviorSubject와 달라진건 초기값이 주어지지 않고 버퍼사이즈가 주어진다는점외에는 모두 이전과 비슷하게 동작한다. 다만 몇가지 눈여겨볼점들을 살펴보자

- 버퍼사이즈만큼의 replay가 이루어지게 된다.
- error가 주어졌을때 종료된뒤 구독이 일어난 구독자에게 종료이벤트(error 또는 completed)만 재방출되는게 아닌 남아있는 버퍼의 내용도 같이 방출된다는게 큰 차이점이다.

버퍼의 방출을 막고싶다면 실제로 사용하지는 않는 방법이지만 anError를 서브젝트에 넘겨준후에 .dispose로 서브젝트를 종료시키면 된다. (실제로는  subscribe 뒤에 .disposed를 붙여 자동으로 메모리해제되도록 설정한다.)

# Relay

이전에 언급했듯이 릴레이는 서브젝트의 replay behavior를 유지한채로 서브젝트를 래핑한다. 또한 릴레이에서는 값을 추가할때는 accept 메소드로만 가능하다. onNext는 불가능하며 릴레이가 오직 값만을 accept하기 때문에 그렇다. (error나 complet를 받을수는 없다.)

릴레이에는 두가지 종류가 있다 publishSubject를 래핑하는 publishRelay, behaivorSubject를 래핑하는 behaviorRelay, 먼저 publish Subject를 살펴보자

## PublishRelay

퍼블리쉬 서브젝트를 래핑하며 completed, error이벤트를 못받는다는 점외에는 차이가 없다.

```swift
example(of: "PublishRelay") {
  let relay = PublishRelay<String>()
  let disposeBag = DisposeBag()

	relay.accept("Knock knock, anyone home?")

	relay
	  .subscribe(onNext: {
			print($0) })
	  .disposed(by: disposeBag)
	relay.accept("1")
	--- Example of: PublishRelay ---
	1

	relay.accept(MyError.anError)
	// 컴파일러 에러
	relay.onCompleted()
	// 컴파일러 에러
}
```

- 이벤트를 onNext가 아닌 accept로 받는다.
- error, completed이벤트에 대해 컴파일러 에러를 낸다.

## BehaviorRelay

비헤이비어 릴레이는 비헤이비어 서브젝트를 래핑하며 퍼블리쉬 릴레이와 마찬가지로 error와 completed이벤트를 받아 종료될수 없다. 다만 한가지 특별한 기능이 있는데 언제든지 마지막 엘리먼트의 값이 무엇인지 가져올수 있다.

```swift
example(of: "BehaviorRelay") {
  // 1
  let relay = BehaviorRelay(value: "Initial value")
  let disposeBag = DisposeBag()
// 2
  relay.accept("New initial value")
// 3
  relay
    .subscribe {
      print(label: "1)", event: $0)
    }
    .disposed(by: disposeBag)
	--- Example of: BehaviorRelay ---
	1) New initial value
	
	// 1
	relay.accept("1")
	1) 1

	// 2
	relay
	  .subscribe {
	    print(label: "2)", event: $0)
	  }
	  .disposed(by: disposeBag)
	2) 1
	// 3
	relay.accept("2")
	1) 2
	2) 2

	print(relay.value)
  2
}
```

위 코드를 통해 보면 기존과 별차이 점은 보이지 않는다 릴레이 답게 onNext가 아닌 accept로 값을 넘겨주며 그외 동작은 기존 relaySubject와 동일하다 한가지 특별한건 relay의 vlaue 프로퍼티에 접근해 가장 마지막 엘리먼트, 가장 최신의 엘리먼트 값에 접근할수 있다.

- accept를 이용해서 값을 추가
- .value프로퍼티를 이용해 가장 마지막 엘리먼트 접근 가능

### 정리

비헤이비어 릴레이는 굉장히 다재다능하며 실제 코딩에서 많이 쓰인다. 기존 서브젝트처럼 구독후 새로운 next이벤트가 방출될때마다 반응이 가능하며 최신값을 따로 구독하지 않더라도 value프로퍼티를 이용해 접근가능하다. 자세한 쓰임새는 이챕터의 두번째 챌린지를 참고하길 바란다!