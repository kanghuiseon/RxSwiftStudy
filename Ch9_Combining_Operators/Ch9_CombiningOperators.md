# Ch9 Combining Operators

## Prefixing and concatenating
### startWith
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/0.png" height = 200>

* 만약 현재 state에서 항상 어떤 이벤트를 방출하는 것으로 이벤트를 시작하고 싶다면, 접두사를 붙여준다.
* 그림에서와 같이, 옵저버블이 요소를 방출하기 전에 1이라는 요소를 가진 next 이벤트를 앞부분에 추가하여 먼저 방출한 후, 다음 이벤트들을 방출한다.
* 
```swift
example(of: "startWith") {
  // 1
  let numbers = Observable.of(2, 3, 4)
  
  // 2
  let observable = numbers.startWith(1)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
}
```

* 예제를 보면, 2, 3, 4를 방출시키는 observable인 numbers를 우선 생성한다.
* 그리고 나서, startWith(1) 연산자를 추가하여 1을 방출하는 이벤트를 앞에서 생성한 numbers내의 이벤트 앞부분에 추가한다.
* startWith는 기본값이나 시작값을 지정할 때 사용한다.
* print -> 1 2 3 4

```swift
let numbers = [1, 2, 3, 4, 5]


Observable.from(numbers)
    .startWith(0)
    .startWith(-1)
    .startWith(-2)
    .startWith(-3)
    .subscribe{ print($0) }
    .disposed(by: bag)
    
print -> 
next(-3)
next(-2)
next(-1)
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
```

* 또한 startWith를 여러개 추가하여 이벤트를 출력해보면, startWith를 추가한 순서의 역순으로 이벤트를 출력하는 것을 볼 수 있다.
* 그 이유는 위에서부터 차례대로 기존의 numbers의 맨 앞에 0을 추가하고, 추가한 항목 앞에 다시 -1을 추가하고, 그 앞에 -2를 추가하고 그 앞에 -3을 추가해서, 역순으로 출력되는 것이다.

<br/>
<br/>

### Observable.concat
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/1.png" height = 200>

* concat메소드는 그림과 같이 두 개의 observable을 연결할 때 사용한다.
```swift
example(of: "Observable.concat") {
  // 1
  let first = Observable.of(1, 2, 3)
  let second = Observable.of(4, 5, 6)
  
  // 2
  let observable = Observable.concat([first, second])
  
  observable.subscribe(onNext: { value in
    print(value)
  })
}

print ->
1
2
3
4
5
6
```
* 출력 값을 보면, first observable의 값과, second observable의 값이 함께 출력된다.
* Observable.concat 연산자는 연결할 observable의 배열을 파라미터값을 받는다.
* 만약 내부의 어떤 Observable이 error를 방출하면, 그 즉시 연결한 observable이 error를 방출하고 종료된다.

<br/>
<br/>

### concat
```swift
example(of: "concat") {
  let germanCities = Observable.of("Berlin", "Münich", "Frankfurt")
  let spanishCities = Observable.of("Madrid", "Barcelona", "Valencia")
  
  let observable = germanCities.concat(spanishCities)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
  
print ->
Berlin 
Münich
Frankfurt
Madrid
Barcelona
Valencia
}
```
* Observable.concat과 같이 Observable의 class 메소드가 아니라, instance 메소드로 사용할 수도 있다.
* 이것도 Observable.concat과 마찬가지로, germanCities가 먼저 방출되고 이후에 spanishCities가 방출된다.
* 무조건 같은 타입의 요소를 방출하는 observable을 연결해야만 한다. 그렇지 않으면 error가 발생한다.


<br/>
<br/>

### concatMap
* concatMap은 앞에서 배운 flatMap가 관련이 있다. 
```swift
example(of: "concatMap") {
  // 1
  let sequences = [
    "German cities": Observable.of("Berlin", "Münich", "Frankfurt"),
    "Spanish cities": Observable.of("Madrid", "Barcelona", "Valencia")
  ]

  // 2
  let observable = Observable.of("German cities", "Spanish cities")
    .concatMap { country in sequences[country] ?? .empty() }

  // 3
  _ = observable.subscribe(onNext: { string in
      print(string)
    })
}

print ->
Berlin
Münich
Frankfurt
Madrid
Barcelona
Valencia
```
* concatMap은 closure가 만드는 각각의 sequence가 합쳐지는 것이 다음 sequence를 구독하기전에 완료되는것을 보장한다.
* 예시를 보면, 
1. 우선 german과 spanish의 도시 이름을 만드는 sequences를 생성하고,
2. country의 이름에 해당하는 sequence의 요소들을 방출한다.
3. 구독자를 추가하여, 방출한 요소를 출력하고, 다음 국가를 확인한다.


<br/>
<br/>

## Merging
* RxSwift는 sequences를 combine하는 여러가지 방법을 제공한다.
### merge
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/2.png" height = 250>

* 그림과 같이 여러 Observable 항목들이 방출하는 항목들을 하나의 Observable에서 방출하도록 병합한다.
```swift
example(of: "merge") {
  // 1
  let left = PublishSubject<String>()
  let right = PublishSubject<String>()
  
  //2
  let source = Observable.of(left.asObservable(), right.asObservable())
  
  // 3
  let observable = source.merge()
  _ = observable.subscribe(onNext: { value in
      print(value) })

 // 4
  var leftValues = ["Berlin", "Munich", "Frankfurt"]
  var rightValues = ["Madrid", "Barcelona", "Valencia"]
  repeat {
      switch Bool.random() {
      case true where !leftValues.isEmpty:
          left.onNext("Left:  " + leftValues.removeFirst())
      case false where !rightValues.isEmpty:
          right.onNext("Right: " + rightValues.removeFirst())
      default:
          break
      }
  } while !leftValues.isEmpty || !rightValues.isEmpty
  
  // 5
  left.onCompleted()
  right.onCompleted()
  
print ->
Right: Madrid
Left:  Berlin
Right: Barcelona
Right: Valencia
Left:  Munich
Left:  Frankfürt
}
```
* 코드를 번호 순서대로 살펴보면,
1. 두개의 PublishSubject 인스턴스를 생성한다.
2. 위의 두 개의 PublishSubject의 asObservable()을 이용하여, observable로 생성하고나서, 이 observable을 파라미터로 받는 source observable을 생성한다.
3. **source.merge()** 를 통해서 파라미터로 넘긴 두 개의 observable을 병합한다. 그리고 나서 구독자를 추가하여 각 observable의 요소를 출력하도록 한다.
4. 랜덤으로 true, false값을 생성해서, 만약 true라면 leftValues의 첫번째 요소를 방출하고, 배열에서 제거한다. 또한 만약 false라면, rightValues의 첫번째 요소를 방출하고, 배열에서 제거한다.
5. left.onCompleted(), right.onCompleted()를 통해 완료 이벤트를 구독자에게 방출한다.

* merge에서 병합된 observable들이 모두 onCompleted 이벤트를 방출해야 구독자에게 completed 메세지를 전달한다.
* 만약 left만 onCompleted()를 방출했다면 구독자에게는 completed메세지가 전달되지 않는다.
* 병합 대상 중 하나라도 에러를 방출하면 그 즉시 구독자에게 에러 이벤트를 전달하고, 더 이상 이벤트를 전달하지 않는다.

<br/>

### merge(maxConcurrent: )
* 위의 merge 연산자는 병합할 수 있는 observable의 수가 제한되지 않는다.
* 만약 제한하고 싶다면 merge(maxConcurrent: )연산자를 이용한다.
```swift
let bag = DisposeBag()
let fruits = BehaviorSubject<String>(value: "apple")
let animals = BehaviorSubject<String>(value: "lion")
let colors = BehaviorSubject<String>(value: "red")
//1
Observable.of(fruits, animals, colors)
    .merge(maxConcurrent: 2)
    .subscribe{ print($0)}
    .disposed(by: bag)
    // 2
    fruits.onCompleted()

print ->
//1
next(apple)
next(lion)

//2
next(apple)
next(lion)
next(red)
```
* max값을 2개로 지정했기 때문에 fruits, animals의 요소만 방출한다.
* merge 연산자는 마지막의 colors를 큐에 저장해놨다가 구독중인 observable가 completed 이벤트를 방출하면 큐의 순서대로 병합 대상에 추가한다.
* 따라서 위의 코드에서 fruits.onCompleted() 이후에 출력값인 //2를 보면, next(red)가 출력된것을 볼 수 있다.

<br/>
<br/>

## Combining elements
### combineLatest
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/3.png" height = 200>

* 사진에서 보면, left 에서 1을 방출했지만 아무것도 방출되지 않았다. 하지만 right에서 4를 방출하자마자 1과4가 결합된 형태의 이벤트가 방출되었다. 
* 또한 이후에 right에서 5를 방출하자 left에서 가장 최근에 방출된 요소인 1과 결합하여, 이벤트가 방출된다.
* 이와 같이, 두 개의 Observable에서 가장 최근에 방출된 이벤트와 결합을 시킬때 사용한다.

```swift
example(of: "combineLatest") {
  let left = PublishSubject<String>()
  let right = PublishSubject<String>()
  
  // 1
  let observable = Observable.combineLatest(left, right) {
    lastLeft, lastRight in
        "\(lastLeft) \(lastRight)"
    }

  _ = observable.subscribe(onNext: { value in
    print(value)
    })
    // 2
  print("> Sending a value to Left")
  left.onNext("Hello,")
  print("> Sending a value to Right")
  right.onNext("world")
  print("> Sending another value to Right")
  right.onNext("RxSwift")
  print("> Sending another value to Left")
  left.onNext("Have a good day,")
  
  
  left.onCompleted()
  right.onNext("Hee")
  
  
  right.onCompleted()
  
  
print -> 
> Sending a value to Left
> Sending a value to Right
Hello, world
> Sending another value to Right
Hello, RxSwift
> Sending another value to Left
Have a good day, RxSwift
Have a good day, Hee
}
     
```
* 코드를 살펴보면, PublishSubject인스턴스인 left, right를 생성하고, Observable.combineLatest를 통해 하나로 결합한다.
* combineLatest의 클로저 내부에서, 현재 방출된 left의 최신 이벤트와 right의 최신 이벤트를 출력하도록 한다.
* 2번에서, left.onNext("Hello, ")를 방출했더니 아무것도 출력되지 않는다.
* 이후 right.onNext("world")를 방출했더니, Hello, world가 출력된다.
* 다시 right.onNext("RxSwift")를 방출했더니, left의 가장 최근에 방출된 Hello, 와 right의 RxSwift가 결합된 형태인 Hello, RxSwift가 출력되는 것을 볼 수 있다.
* 만약 둘 중 하나가 onCompleted 이벤트를 방출하고, 나머지가 onNext 이벤트를 방출하면 completed된 observable의 가장 최근 이벤트 요소를 방출한다. 따라서, left의 가장 최근 요소인 Have a good day와 right의 최근 요소인 Hee가 함께 결합되어 출력된다.
* 둘 다 completed가 되면 그제서야 구독자에게 completed이벤트를 전달한다.
* 만약 하나라도 error 이벤트를 방출한다면 바로 구독자에게 error이벤트를 전달하고 종료한다.
* map(_:)과 같이, combineLatest(_:_:resultSelector)은 closure의 return type을 갖는 observable을 생성한다. 

<br/>
<br/>

### combineLatest(_:_:resultSelector)
* 이 연산자는 2~8개의 observable sequences를 파라미터로 받는다. 각 observable의 타입은 같을 필요는 없다.
```swift
example(of: "combine user choice and value") {
  let choice: Observable<DateFormatter.Style> = Observable.of(.short, .long)
  let dates = Observable.of(Date())
  
  let observable = Observable.combineLatest(choice, dates) {
    format, when -> String in
    let formatter = DateFormatter()
    formatter.dateStyle = format
    return formatter.string(from: when)
  }
  
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
}
print ->
3/23/21
March 23, 2021
```
* 위의 예제는 사용자가 세팅을 변경할때마다 화면의 값을 자동으로 업데이트해준다. 

<br/>
<br/>

### combineLatest(_:_:resultSelector)
* observable의 배열을 파라미터로 받을 수도 있다. 배열이기 때문에 Observable들의 타입은 동일해야한다.
* 유동적으로 사용할 수 없기 때문에 잘 사용되지는 않는다.
```swift
// 1
let left = PublishSubject<String>()
let right = PublishSubject<String>()

let observable = Observable.combineLatest([left, right]) {
    strings in strings.joined(separator: " ")
}
_ = observable.subscribe(onNext: { value in
  print(value)
})
left.onNext("Hello, ")
right.onNext("Hee")

print ->
Hello, Hee
```

<br/>
<br/>

### zip
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/4.png" height = 200>

* zip연산자는 인덱스를 맞춘다고 생각하면 된다. 그림에서와 같이 left의 첫번째 인덱스인 sunny는 right이 이벤트를 방출할때까지 기다렸다가 Lisbon과 함께 결합되어서 방출된다.
* 또한 left의 두번째 인덱스인 cloudy는 right의 두번째 인덱스인 London과 함께 결합되어서 나온다.
* 이와 같이 같은 인덱스의 요소가 방출되기 전까지 기다리고 있다가 방출된다.

```swift
example(of: "zip") {
  enum Weather {
    case cloudy
    case sunny
  }
  let left: Observable<Weather> = Observable.of(.sunny, .cloudy, .cloudy, .sunny)
  let right = Observable.of("Lisbon", "Copenhagen", "London", "Madrid", "Vienna")
  let observable = Observable.zip(left, right) { weather, city in
      return "It's \(weather) in \(city)"
    }
    _ = observable.subscribe(onNext: { value in
      print(value)
    })
  }

print -> 
——— Example of: zip ———
It's sunny in Lisbon
It's cloudy in Copenhagen
It's cloudy in London
It's sunny in Madrid
```
* zip 연산자는 left, right 연산자를 구독하고, 같은 인덱스의 요소들과 함께 결합되어서 출력된다.
* right의 마지막 Vienna는 방출되지 않았다. 그 이유는 Vienna와 동일한 인덱스의 요소가 방출되지 않았기 때문이다.
* 이렇게 인덱스를 이용해 이벤트를 방출하는 것을 **Indexed Sequencing** 이라고 한다.

<br/>
<br/>

## Triggers
* 어떤 액션을 취했을때, 원하는 데이터를 가져오는 기능과 같이, trigger action이 필요할때에는 아래의 연산자들을 사용한다.
<br/>

### withLatestFrom
<img src="https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/5.png" height=200>

*  그림에서처럼 textfield에서 텍스트를 검색하고, 버튼을 탭하면, textfield의 가장 최근 데이터를 가져오도록할 때 이 연산자를 사용한다.

```swift
example(of: "withLatestFrom") {
  // 1
  let button = PublishSubject<Void>()
  let textField = PublishSubject<String>()
  
  // 2
  let observable = button.withLatestFrom(textField)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
  
  // 3
  textField.onNext("Par")
  textField.onNext("Pari")
  textField.onNext("Paris")
  button.onNext(())
  button.onNext(())
}

print ->
Paris
Paris
```
* 1. 코드에서와 같이 Void 타입의 button과 String 타입의 textField를 각각 생성한다.
* 2. button은 textfield의 값을 가져오기 위한 trigger역할이고, textfield는 data역할을 한다.
* 2. 형식은 trigger.withLatestFrom(data)이다.
* 3. trigger가 next 이벤트를 방출할때까지 data는 이벤트를 방출하지 않는다.
* 3. trigger가 next 이벤트를 방출하는 순간, textField에서 가장 최근에 방출된 이벤트를 방출하여 구독자에게 전달을 해준다.
* print를 보면 button에서 next 이벤트를 방출할때마다 textfield의 가장 최근 이벤트 요소를 출력하는 것을 볼 수 있다.
* 만약 textField.onCompleted()를 해도, 구독자에게 전달되지 않는다. 여전히 button.onNext(())를 하면 Paris가 출력된다.
* 하지만 button.onCompleted() 이후에는 바로 구독자에게 completed이벤트가 전달되고 종료된다.
* textField observable이 에러 이벤트를 방출할때에는 구독자에게 바로 전달이 되고 종료된다.
<br/>
<br/>

### sample
<img src="https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/6.png" height=200>

* sample 연산자는 withLatestFrom연산자와 비슷하지만, 다른 점이 있다. 사진에서 처럼, tap을 여러번 해도 Paris가 한번만 출력된다.
* 이와 같이, trigger에서 onNext 이벤트를 방출할때마다 data에서는 최신 observable을 **한번만** 방출한다.
```swift
let observable = textField.sample(button)’
```
* **또한 withLatestFrom 연산자와는 다르게, trigger와 data의 위치가 다르다.**
* 만약 withLatestFrom 연산자를 sample연산자와 동일하게 사용하고 싶다면 distinctUntilChanged() 연산자를 추가해준다.

<br/>
<br/>

## Switches
RxSwift에서는 "switching" 연산자인 amb(_:)와 switchLatest()연산자를 제공한다.
### amb
* 2개 이상의 observables중에서 가장 먼저 next 이벤트를 전달하는 Observable을 구독하고, 나머지는 모두 무시한다.
<img src="https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/7.png" height=200>

* 사진에서와 같이, right이 가장 먼저 next 이벤트를 방출했기 때문에 left observable에서 방출되는 이벤트들은 모두 무시된다.
```swift
example(of: "amb") {
  let left = PublishSubject<String>()
  let right = PublishSubject<String>()
  
  // 1
  let observable = left.amb(right)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
  
  // 2
  left.onNext("Lisbon")
  right.onNext("Copenhagen")
  left.onNext("London")
  left.onNext("Madrid")
  right.onNext("Vienna")
  
  left.onCompleted()
  right.onCompleted()
}

print ->
Lisbon
London
Madrid
```
* amb 연산자는 left와 right observables를 구독하고, 둘 중 하나가 이벤트를 방출할때까지 기다린다.
* 만약 한쪽이라도 방출을 하면, 나머지 하나의 구독을 취소한다.

<br/>
<br/>

### switchLatest
<img src="https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/8.png" height=200>

* switchLatest는 가장 최근에 구독한 옵저버블이 방출한 이벤트를 구독자에게 전달한다.

```swift
example(of: "switchLatest") {
  // 1
  let one = PublishSubject<String>()
  let two = PublishSubject<String>()
  let three = PublishSubject<String>()
  
  let source = PublishSubject<Observable<String>>()
  
  // 2
  let observable = source.switchLatest()
  let disposable = observable.subscribe(onNext: { value in
      print(value)
    }) 
    
    // 3
  source.onNext(one)
  one.onNext("Some text from sequence one")
  two.onNext("Some text from sequence two")
  
  source.onNext(two)
  two.onNext("More text from sequence two")
  one.onNext("and also from sequence one")
  
  source.onNext(three)
  two.onNext("Why don't you see me?")
  one.onNext("I'm alone, help me")
  three.onNext("Hey it's three. I win.")
  
  source.onNext(one)
  one.onNext("Nope. It's me, one!")
  disposable.dispose()
}

print ->
——— Example of: switchLatest ———
Some text from sequence one
More text from sequence two
Hey it's three. I win.
Nope. It's me, one!
```
* 1. 우선, 세 개의 PublishSubject 인스턴스를 생성하고, Observable<String> 타입의 PublishSubject도 생성한다.
* 2. source에 switchLatest 연산자를 추가하고, 구독자를 추가한다.
* 3. 출력되는 결과를 살펴보면 source가 가장 최근에 구독한 observable의 이벤트만을 출력하는 것을 볼 수 있다.
* 3. source가 one을 구독할때에는 one이 방출한 이벤트만을 출력하고, two를 구독하기 시작할때에는 two가 방출한 이벤트만 출력한다. three를 구독할때에도 마찬가지이다. 

<br/>
<br/>

## Combining elements within a sequence
## reduce
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/9.png" height=200>

* 기존의 Swift 문법인 reduce와 비슷하다.
* 사진에서처럼 sequence의 요소들의 누적합을 방출한다.

```swift
example(of: "reduce") {
  let source = Observable.of(1, 3, 5, 7, 9)
  
  // 1
  let observable = source.reduce(0, accumulator: +)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
  
  //2
  let observable2 = source.reduce(0) { summary, newValue in
    return summary + newValue
  }
}
```
* 1. 초기값으로 설정한 0에서부터 source의 요소들을 합한 값이 출력된다.
* 2. 2번은 1번의 다른 형태로, accumulator에 + 클로저를 건네주는 것이 아니라, 클로저를 직접 구현하는 형식이다.

<br/>
<br/>

### scan
<img src = "https://github.com/kanghuiseon/RxSwiftStudy/blob/master/Ch9_Combining_Operators/Resource/10.png" height=200>

* scan 연산자는 reduce와 유사하지만 다른 점이 있다. 
* reduce연산자는 최종적으로 누적된 값을 출력하지만 scan 연산자는 값을 누적시킬때마다를 출력한다.
```swift
example(of: "scan") {
  let source = Observable.of(1, 3, 5, 7, 9)
  
  let observable = source.scan(0, accumulator: +)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })
  
print ->
——— Example of: scan ———
1
4
9
16
25
}
```
* 위의 코드에서 출력값을 보면, 초깃값 0에서부터 차례대로 더한 값이 각각 출력된 것을 볼 수 있다. 예를 들어, 0+1, 1+3, 4+5, 9+7, 16+9 가 각각 출력되었다.
* 자세한 예제는 Ch20_RxGesture에서 배울 수 있다.

<br/>
<br/>

## Challenge
### Challenge: The zip case
* scan예제에 zip 연산자를 추가해서 현재 값과 총합이 각각 출력되도록 해라
```swift
let bag = DisposeBag()
let source = Observable.of(1, 3, 5, 7, 9)
let observable = source.scan(0, accumulator: +)

Observable.zip(source, observable) {
    "\($0) \($1)"
}
.subscribe(onNext: {
    print($0)
})
.disposed(by: bag)

print -> 
1 1
3 4
5 9
7 16
9 25
```
