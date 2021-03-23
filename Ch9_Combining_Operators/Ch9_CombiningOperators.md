# Ch9 Combining Operators

## Prefixing and concatenating
### startWith
<img src = "0" height = 150>

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

### concat
<img src = "1" height = 150>
