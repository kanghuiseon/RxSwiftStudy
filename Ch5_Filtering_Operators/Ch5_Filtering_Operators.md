# Ch5 Filtering Operators
> 이번 챕터에서는 RxSwift의 filtering operators에 대해서 다룬다. 방출되는 이벤트에 조건을 적용해서 사용한다.
> 조건을 통해 구독자는 필터링된 요소만 받는다.
> swift의 filter(_:) 와 유사하다.

<br/>

## Ignoring operators
### ignoreElements
<img src="0" height=200>

* 모든 next이벤트는 무시하고, completed 와 error 이벤트만 전달한다.

```swift
example(of: "ignoreElements") {
  // 1
  let strikes = PublishSubject<String>()

  let disposeBag = DisposeBag()

  // 2
  strikes
    .ignoreElements()
    .subscribe { _ in
      print("You're out!")
    }
    .disposed(by: disposeBag)
    
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")

strikes.onCompleted()
    
print ->
--- Example of: ignoreElements ---
You're out!
}
```
* 위의 예시에서 onNext이벤트는 구독자에게 전달하지 않고, completed 이벤트만 전달했기 때문에 출력은 한번만 된다.
* ignoreElements 연산자는 observable이 completed or error 이벤트를 통해 종료되었는지 알고싶을때 사용한다.

<br/>
<br/>

### elementAt
<img src = "1" height = 200>

* 만약 특정 위치의 요소를 방출하고 싶다면 이 연산자를 사용한다.
* 사진에서와 같이 elementAt의 파라미터로 전달하면 두번째 인덱스의 이벤트만 구독자에게 전달된다. (인덱스는 0부터 시작하니까!)

```swift
example(of: "elementAt") {

  // 1
  let strikes = PublishSubject<String>()

  let disposeBag = DisposeBag()

  //  2
  strikes
    .elementAt(2)
    .subscribe(onNext: { _ in
      print("You're out!")
    })
    .disposed(by: disposeBag)
}

strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")

print -> 
--- Example of: elementAt ---
You're out!
```
* 예시를 보면, 두번째 인덱스에 위치한 세번째인 next 이벤트만을 구독자에게 전달하기 때문에, 출력도 한번만 된다.
* element(at: )의 흥미로운 사실은, 해당 인덱스의 이벤트가 방출되자마자 구독은 종료된다.

<br/>
<br/>

### filter
<img src = "2" height=200>

* 위의 사진에서처럼 filter연산자는 원하는 조건에 해당하는 이벤트만 방출한다.
* 만약 조건이 true라면 구독자에게 전달하고, false라면 전달하지 않는다.
* 그림에서는 3보다 작은 요소만을 방출하기 때문에 3은 구독자에게 전달되지 않는다.

```swift
example(of: "filter") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .filter { $0.isMultiple(of: 2) }
    // 3
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: filter ---
2
4
6
```
* 위의 코드에서는 짝수인 요소만을 가진 이벤트만 구독자에게 방출하도록 조건을 설정하였기 때문에, 2,4,6만 출력된다.

<br/>
<br/>

## Skipping operators
<img src = "3" height=200>

* 만약 특정 수의 요소를 무시하고, 그 다음 요소부터 구독자에게 전달하고 싶다면, skip 연산자를 사용한다.
* 사진에서는 파라미터로 2를 전달했으므로, 앞의 1, 2의 요소는 구독자에게 전달되지 않는다.
```swift
example(of: "skip") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of("A", "B", "C", "D", "E", "F")
    // 2
    .skip(3)
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: skip ---
D
E
F
```
* 문자열을 가진 Observable을 생성하고, skip(3) 연산자를 추가해준다. 그리고 나서 출력을 보면, A, B, C가 무시되고 이후의 이벤트만 출력되는것을 볼 수 있다.

<br/>
<br/>

### skipWhile
<img src = "4" height=200>

* skipWhile에서는 조건문이 true를 리턴하는 것은 skip한다. false를 리턴한 순간부터는 skip하지 않고 그대로 구독자에게 방출한다. 
* 예를 들어, 그림에서는 홀수인 1은 true를 리턴한다. 하지만 2는 false를 리턴하기 때문에 이 이후부터는 홀수가 나와도 그대로 방출한다.
```swift
example(of: "skipWhile") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of(2, 2, 3, 4, 4)
    // 2
    .skipWhile { $0.isMultiple(of: 2) }
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: skipWhile ---
3
4
4
```
* 위의 코드에서 skipWhile의 조건을 보면, 홀수가 나타날때까지 즉, false를 리턴할때까지 요소들을 모두 skip한다.
* 따라서, 출력을 보면, 첫번째 2와 두번째 2는 조건에서 true를 리턴하기 때문에 무시되고, 3은 false를 리턴하기 때문에, 이후의 4들이 조건을 만족한다해도 그대로 구독자에게 전달된다.

<br/>
<br/>

### skipUntil
<img src="5" height=200>

* 파라미터로 전달된 observable이 next 이벤트를 전달하기 전까지 원래의 observable이 전달하는 이벤트는 모두 무시한다.

```swift
example(of: "skipUntil") {
  let disposeBag = DisposeBag()

  // 1
  let subject = PublishSubject<String>()
  let trigger = PublishSubject<String>()

  // 2
  subject
    .skipUntil(trigger)
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
  subject.onNext("A")
  subject.onNext("B")

  trigger.onNext("X")
  subject.onNext("C")
  
print ->
--- Example of: skipUntil ---
C
}
```
* 코드에서 보면, skipUntil 파라미터로 trigger subject를 넘겨주었다.
* trigger가 onNext 이벤트를 방출하지 않았기 때문에 바로 위의 subject.onNext("A")와 subject.onNext("B")는 구독자에게 전달되지 않았다. 
* 하지만 trigger가 onNext이벤트를 방출하고나서는, subject.onNext("C") 이벤트는 구독자에게 전달되었다.

<br/>
<br/>

## Taking operators
* taking은 skip과는 반대이다. 무시하는 것이 아니라, 원하는만큼만 이벤트를 방출한다.

### take
<img src = "6" height=200>

* take 연산자는 파라미터로 전달된 정수만큼만 요소를 방출한다.
* 사진에서 보는것과 같이 2개의 이벤트만 구독자에게 전달된다.

```swift
example(of: "take") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .take(3)
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print -> 
--- Example of: take ---
1
2
3
```
* take 연산자에 파라미터로 3을 넘겨주었기 때문에, 1,2,3 만 출력된 것을 볼 수 있다.


<br/>
<br/>


### takeWhile
<img src="7" height=200>

* skipWhile과는 반대로, true를 리턴하면 구독자에게 전달하고, false를 리턴한 순간부터는 더 이상 요소를 방출하지 않는다.

```swift
example(of: "takeWhile") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of(2, 2, 4, 4, 6, 6)
    // 2
    .enumerated()
    // 3
    .takeWhile { index, integer in
      // 4
      integer.isMultiple(of: 2) && index < 3
    }
    // 5
    .map(\.element)
    // 6
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: takeWhile ---
2
2
4
```
* enumerated 연산자를 사용하면 방출하는 이벤트의 index값을 알 수 있다.
* takeWhile의 조건식을 보면 integer가 짝수이고 index가 3보다 작을 경우에 true를 리턴한다.
* 따라서 index가 2까지인 요소만 방출하고 이후의 요소들은 false를 리턴하기 때문에 더 이상 방출하지 않는다.

<br/>
<br/>

### takeUntil
<img src="8" height=200>

* takeUntil은 조건문의 식이 true를 return 할때까지 요소를 방출한다.
* 또한 behavior argument를 설정하는데, .inclusive는 조건을 만족하는 마지막 요소를 포함시킨다는 의미이고, .exclude는 제외시킨다는 의미이다.

```swift
example(of: "takeUntil") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of(1, 2, 3, 4, 5)
    // 2
    .takeUntil(.inclusive) { $0.isMultiple(of: 4) }
    .subscribe(onNext: {
      print($0)
    })
  .disposed(by: disposeBag)
}
print ->
--- Example of: takeUntil ---
1
2
3
4
```
* 현재는 .inclusive로 설정했기 때문에 조건을 만족하는 4가 포함되어 있지만, .exclusive로 설정하면 1, 2, 3만 출력된다.

<br/>
<br/>

<img src="9" height=200>

* takeUntil은 skipUntil과 마찬가지로, trigger observable과 함께 작동할 수도 있다.
* trigger observable이 onNext 이벤트를 방출하기 전까지 요소를 방출한다.

```swift
example(of: "takeUntil trigger") {
  let disposeBag = DisposeBag()

  // 1
  let subject = PublishSubject<String>()
  let trigger = PublishSubject<String>()

  // 2
  subject
    .takeUntil(trigger)
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)

  // 3
  subject.onNext("1")
  subject.onNext("2")
  
  trigger.onNext("X")

  subject.onNext("3")
}
print ->
--- Example of: takeUntil trigger ---
1
2
```
* trigger observable이 onNext("X") 이벤트를 방출하고나서, subject가 onNext("3")이벤트를 방출한다.
* 하지만 출력을 보면, 3은 출력되지 않고, trigger의 onNext 이벤트 방출 전까지의 요소들만 출력이 되었다.

<br/>
<br/>

## Distinct operators
* 다음의 연산자들은 연속적인 요소들에 대해 중복을 방지하도록 한다.
<br/>

### distinctUntilChanged
<img src = "10" height=200>

* 그림에서 처럼 2가 연속으로 나왔기 때문에 첫 2 이후의 2는 구독자에게 전달하지 않는다.

```swift
example(of: "distinctUntilChanged") {
  let disposeBag = DisposeBag()

  // 1
  Observable.of("A", "A", "B", "B", "A")
    // 2
    .distinctUntilChanged()
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: distinctUntilChanged ---
A
B
A
```
* 위의 코드의 출력을 보면, 중복된 "A", "B"가 구독자에게 전달되지 않은 것을 볼 수 있다.
* 마지막의 "A"는 중복된 값이긴 하지만 연속적으로 중복되지 않았기 때문에 그대로 출력된다.

<br/>
<br/>

### distinctUntilChanged(_:)
<img src="11" height=200>

* 만약 사진에서처럼 custom하게 조건을 제공하고 싶다면, 이 연산자를 사용한다.

```swift
example(of: "distinctUntilChanged(_:)") {
  let disposeBag = DisposeBag()
  
  // 1
  let formatter = NumberFormatter()
  formatter.numberStyle = .spellOut

  // 2
  Observable<NSNumber>.of(10, 110, 20, 200, 210, 310)
    // 3
    .distinctUntilChanged { a, b in
      // 4
      guard
        let aWords = formatter
          .string(from: a)?
          .components(separatedBy: " "),
        let bWords = formatter
          .string(from: b)?
          .components(separatedBy: " ")
        else {
          return false
      }

      var containsMatch = false

      // 5
      for aWord in aWords where bWords.contains(aWord) {
        containsMatch = true
        break
      }
      return containsMatch
    }
    // 6
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
}

print ->
--- Example of: distinctUntilChanged(_:) ---
10
20
200
```
* 위의 코드에서
1. 숫자의 스펠링 문자열을 만들어주는 formatter를 생성한다.
2. 10, 110, 20, 200, 210, 310의 요소를 가지는 observable을 생성한다.
3. distinctUntilChanged(_:) 연산자를 사용해서, 커스텀한 조건문을 만들기 위해 클로저를 파라미터로 넘긴다.
4. 각 스펠링을 띄어쓰기 단위로 나누고, aWords, bWords에 각각 저장한다. (예를 들어, 110의 경우에는 "one", "hundred", "ten"으로 나뉜다.)
5. aWords내의 각각의 요소들이 bWords에 포함되어있는지 체크하고, 포함되어있다면 containMatch를 true로 변경하고 리턴한다.
5. 만약 true가 리턴되었다면 둘중 첫번째 요소만 구독자에게 전달하고 두번째 요소는 전달하지 않는다.
5. 또한 비교에 있어서, 만약 10과 110을 비교해서 10만 출력되었다면, 그 다음의 비교는 10과 20으로 진행한다. (10과 110이 중복되어 110이 무시되서 10이랑 비교가 되는듯? 하다.)
6. 출력값을 보면 10, 110 중에서 둘 다, "ten"을 포함하므로 **10**이 선택되고, 20은 10과 중복되는 것이 없으므로, 그대로 출력하고, 20과 200도 중복된것이 없으므로 **200**출력, 200, 210은 둘 다, "two", "hundred"를 포함하므로 출력 X, 또한 200과 310에서도 "hundred"가 포함되어있으므로 310도 출력하지 않는다.


<br/>
<br/>


## Challenge
* 이번 챌린지에서는 다음의 filter operators를 사용한다.
1. phone number는 0으로 시작하면 안된다. - skipWhile
2. 각 input은 single-digit number여야 한다. 10이상 불가능 - filter
3. U.S. 전화번호는 10 numbers로 제한한다. - take, toArray
<br/>

```swift
let disposeBag = DisposeBag()

let contacts = [
"603-555-1212": "Florent",
"212-555-1212": "Shai",
"408-555-1212": "Marin",
"617-555-1212": "Scott"
]

func phoneNumber(from inputs: [Int]) -> String {
var phone = inputs.map(String.init).joined()

phone.insert("-", at: phone.index(
  phone.startIndex,
  offsetBy: 3)
)

phone.insert("-", at: phone.index(
  phone.startIndex,
  offsetBy: 7)
)

return phone
}

let input = PublishSubject<Int>()

// Add your code here
input
    .skipWhile{ number in
        number == 0
    }
    .filter{ number in
        number < 10
    }
    .take(10)
    .toArray()
    .subscribe(onSuccess: {
        let phone = phoneNumber(from: $0)
        if let contact = contacts[phone] {
          print("Dialing \(contact) (\(phone))...")
        } else {
          print("Contact not found")
        }
    })
    .disposed(by: disposeBag)

input.onNext(0)
input.onNext(603)

input.onNext(2)
input.onNext(1)

// Confirm that 7 results in "Contact not found",
// and then change to 2 and confirm that Shai is found
input.onNext(2)

"5551212".forEach {
if let number = (Int("\($0)")) {
  input.onNext(number)
}
}

input.onNext(9)

print ->
Dialing Shai (212-555-1212)...
```

