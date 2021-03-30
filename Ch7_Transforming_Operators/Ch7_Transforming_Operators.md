# Ch.7 Transforming Operators

> RxSwift에서 가장중요한 오퍼레이터중 하나 : transforming operators에 대해 배웁니다. 이름에서 느껴지는것처럼 observable로 부터오는 데이터를 subscriber가 사용할수 있도록 변환하는 역할을 합니다. swift의 map과 flatmap을 생각하면 좋습니다.

# Transforming Operators: 변환연산자

`toArray`, `map`, `enumarated` 와 같은 변환연산자에 대해 배웁니다.

위에서 설명한것과 같이 각각 observable의 element에 대해 특정 변환을 수행하고 이를 다시 subscriber에게 전달해 줍니다.

- toArray : observable의 독립적인 element를 하나의 array로 묶어 subscriber로 전달해준다.
- map : 각각의 observable요소에 특정 변환을 수행해준다. swift의 map과 같다.
- compactMap :observalbe의 요소중 nil값을 가진것을 거르고 요소에 특정 변환을 수행해준다. swift의               compactmap과 같다.

## 1. toArray

observable이 complete될때 sequence의 요소들을 하나의 array로 변환시킨다. 다시 말해 toArray 오퍼레이터는 Single이다. 값을 가지는 success이벤트와 error를 가지는 error이벤트 밖에 존재 하지 않는다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/1.%20toArray.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/1.%20toArray.png?raw=true)

```swift
example(of: "toArray") {
  let disposeBag = DisposeBag()
    // 1
  Observable.of("A", "B", "C")
    // 2
    .toArray()
    .subscribe(onSuccess: {
print($0) })
    .disposed(by: disposeBag)

}

// print 
--- Example of: toArray ---
["A", "B", "C"]	
```

- observable이 complete될때 elements들의 array를 만들어 보내기 때문에 subscribe에서 onSuccess로 받는것을 알수 있다.

## 2. map

스위프트의 map과 동일하며 observable대상이라는 점외에는 동일하다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/2.%20map.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/2.%20map.png?raw=true)

```swift
example(of: "map") {
  let disposeBag = DisposeBag()
// 1
  let formatter = NumberFormatter()
  formatter.numberStyle = .spellOut
// 2
  Observable<Int>.of(123, 4, 56)
    // 3
.map {
      formatter.string(for: $0) ?? ""
    }
    .subscribe(onNext: {
      print($0)
})
    .disposed(by: disposeBag)
}

// print 
--- Example of: map ---
one hundred twenty-three
four
fifty-six
```

### enumarated with map

enumarated와 혼합해서도 사용가능하다. observables의 element index에 따라 다른 연산을 해줄수도 있다.

```swift
example(of: "enumerated and map") {
  let disposeBag = DisposeBag()
// 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .enumerated()
// 3
    .map { index, integer in
      index > 2 ? integer * 2 : integer
    }
// 4
    .subscribe(onNext: {
      print($0)
})
    .disposed(by: disposeBag)
}

// print
--- Example of: enumerated and map ---
1
2
3
8 
10 
12
```

## 3. compactMap

fiter와 map operator의 조합이라고 생각하면 편하다. filter로 nil인것들을 제외시킨후에 map으로 연산을 진행한다. swift의 compactMap과 유사하다.

```swift
example(of: "compactMap") {
  let disposeBag = DisposeBag()
// 1
  Observable.of("To", "be", nil, "or", "not", "to", "be", nil)
    // 2
    .compactMap { $0 }
    // 3
    .toArray()
    // 4
    .map { $0.joined(separator: " ") }
    // 5
    .subscribe(onSuccess: {
print($0) })
    .disposed(by: disposeBag)
}

// print
--- Example of: compactMap ---
To be or not to be
```

# Transforming Inner Observables: 옵저버블속 옵저버블을 변환하기

위에서까지는 일반적인 값을가지는 옵저버블을 대상으로한 오퍼레이터를 다뤄봤다 "그렇다면 옵저버블을 값으로 가지는 옵저버블에 대한 연산은 어떻게 이루어질까?" 

```swift
struct Student {
  let score: BehaviorSubject<Int>
}
```

위와 같은 Student 스트럭트를 엘리먼트를 가지는 옵저버블은 어떨까? 이럴경우에는 옵저버블 안에 옵저버블이 포함되게 된다.

그 대답에 대해서는 `flatMap`과 `flatMapLatest` 를 통해 알아보자. flatMap 관련함수들은 옵저버블속 옵저버블에 접근해 그에 관한 연산을 수행한다.

- flatMap : observable 시퀀스의 각각 요소를 observable 시퀀스로 투영하고, 투영된 옵저버블 시퀀스들을 하나의 옵저버블 시퀀스로 병합한다.
- flatMapLatest : observable 시퀀스의 각각 요소를 observable 시퀀스로 투영하고 이 옵저버블 시퀀스들중 가장  최근의 옵저버블 시퀀스의 값들로만 옵저버블 시퀀스를 구성한다.

명확한정의지만 이해하기는 어려워보인다. 아래 예시를 통해 살펴보면 조금더 쉽게 이해할수 있다.

## 1. flatMap

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/3.%20flatmap.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/3.%20flatmap.png?raw=true)

맨 위줄이 기본 옵저버블 시퀀스이다. 조금다른 기호가 보이는데 O1, O2, O3는 각각 int 1, 2, 3을 값으로가지는 옵저버블을 이라고 생각하면 좋을것 같다. 아래서 예제를 통해 다시볼떄 이해하면 좋다.

그이후 flatMap을 통해 O1, O2, O3가 각각 새로운 옵저버블로 생성되고 있다. (중간 세개의  마블다이어그램이 투영된 각각의 옵저버블 시퀀스이다.)

마지막에 결과적으로 투영된 시퀀스들이 하나의 옵저버블시퀀스로 병합되어 구성된다.

```swift
example(of: "flatMap") {
  let disposeBag = DisposeBag()
// 1
  let laura = Student(score: BehaviorSubject(value: 80))
  let charlotte = Student(score: BehaviorSubject(value: 90))
// 2
  let student = PublishSubject<Student>()
// 3
  student
    .flatMap {
$0.score }
// 4
    .subscribe(onNext: {
      print($0)
})
    .disposed(by: disposeBag)

student.onNext(laura)
--- Example of: flatMap ---
80

laura.score.onNext(85)
85

student.onNext(charlotte)
90

laura.score.onNext(95)
95

charlotte.score.onNext(100)
100
}

// print
--- Example of: flatMap ---
80
85
90
95
100
```

마지막에 Print로 출력되는게 병합된 하나의 옵저버블 시퀀스라 생각하면 좋을것 같다. 

- 옵저버블을 엘리먼트로 가지는 옵저버블, 마블다이어그램의 첫 줄이 student이다.
- laura, charlotte 등이 투영되는 새로운 옵저버블이다. laura와 charlotte의 점수값을 변경해도 그 변경은 모두 투영되게 된다. 마블다이어그램의 중간 3줄.
- print되는 마지막 결과를 병한된 하나의 옵저버블 시퀀스라 생각하면된다.

## 2. flatMapLatest

observable 시퀀스의 각각 요소를 observable 시퀀스로 투영하고 이 옵저버블 시퀀스들중 가장  최근의 옵저버블 시퀀스의 값들로만 옵저버블 시퀀스를 구성한다.

latest란 말이 추가됐다. 무엇이 달라질까? 정의와 함께 아래 마블다이어그램을 참고하면 쉽게 인사이트를 얻을수 있다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/4.%20flatMapLatest.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/4.%20flatMapLatest.png?raw=true)

위의 그림과 비교해 생각해보면 마지막에 병합되는 부분만 달라졌다. 가장최근의 옵저버블 시퀀스의 값들로만 결과 시퀀스가 구성되는것을 확인할수 있다.

O2의 옵저버블 시퀀스의 값에 변화가 일어나도 결과 시퀀스에는 포함되지 않는다. O3가 가장최근 옵저버블 이기 떄문이다.

```swift
example(of: "flatMapLatest") {
  let disposeBag = DisposeBag()
  let laura = Student(score: BehaviorSubject(value: 80))
  let charlotte = Student(score: BehaviorSubject(value: 90))
  let student = PublishSubject<Student>()
  student
    .flatMapLatest {
$0.score }
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)

  student.onNext(laura)
	--- Example of: flatMapLatest ---
	80

  laura.score.onNext(85)
	85

  student.onNext(charlotte)
	90

// 1
  laura.score.onNext(95)
	// 무시 

  charlotte.score.onNext(100)
	100
}

// print
--- Example of: flatMapLatest ---
80
85
90
100
```

마블다이어그램의 흐름과 똑같다.

- 옵저버블을 엘리먼트로 가지는 옵저버블, 마블다이어그램의 첫 줄이 student이다.
- laura, charlotte 등이 투영되는 새로운 옵저버블이다. laura와 charlotte의 점수값을 변경해도 그 변경은 모두 투영되게 된다. 마블다이어그램의 중간 3줄.
- print되는 마지막 결과에서 laura의 스코어를 변경한 observable이 빠졌다. 가장최근의 charlotte만 반영되기 때문이다.

# Observing Events: 옵저버블을 감싸는 옵저버블을 관찰하기

코딩을 하다보면 옵저버블을 옵저버블의 이벤트의 옵저버블로 구성하는 경우가 필요할수도 있다. 즉 옵저버블의 옵저버블을 제어할수 없고 외부적인 error로 인해 전체 옵저버블이 중단되는것을 막고싶을때 이렇게 구성한다.

아래경우에 엘리먼트가되는 옵저버블에서 error 가 발생하면 시퀀스의 나머지 옵저버블 엘리먼트가 종료되는데 이를 피하고 싶을떄 사용하는 방법들을 알아보자

```swift
example(of: "materialize and dematerialize") {
  // 1
  enum MyError: Error {
    case anError
  }
  let disposeBag = DisposeBag()
// 2
  let laura = Student(score: BehaviorSubject(value: 80))
  let charlotte = Student(score: BehaviorSubject(value: 100))
  let student = BehaviorSubject(value: laura)

	let studentScore = student
  .flatMapLatest {
	$0.score }
	// 2
	studentScore
	  .subscribe(onNext: {
	print($0) })
	  .disposed(by: disposeBag)
		--- Example of: materialize & dematerialize ---
		80

	// 3
	  laura.score.onNext(85)
		85

	  laura.score.onError(MyError.anError)
		Unhandled error happened: anError
		// 종료

	  laura.score.onNext(90)
		// 무시
	// 4
	  student.onNext(charlotte)
		// 무시
}

// print
--- Example of: materialize & dematerialize ---
80
85
Unhandled error happened: anError
```

출력결과에서 보이듯이 laura옵저버블에 에러 이벤트를 등록하자 laura를 엘리먼트로 가지는 student옵저버블이 종료되버렸다. 그결과로 뒤에 들어오는 laura의 이벤트나 student 자체에 대한 이벤트 모두 종료되어 실행이 되지 않는다.

- 내부 옵저버블인 laura가 종료되어 90이 출력되지 않음
- 외부 옵저버블인 student가 종료되어 charlotte를 등록해도 100이 출려되지 않음

## 1. Materialize

위와 같이 바깥 옵저버블이 종료되는것을 materialize 오퍼에리이터를 이용해 해결할수 있다. 

- materailize 오퍼레이터는 이벤트를 래핑하는 역할을 한다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/5.%20materialize.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/5.%20materialize.png?raw=true)

```swift
let studentScore = student
  .flatMapLatest {
    $0.score.materialize()
  }
```

studentScore에 materialize 오퍼레이터를 추가하면 studentScore는 더이상 observable<Int>가아닌 래핑되어 observable<Event<Int>>가 되게 되며 에러가 발생하더라도 외부 옵저버블 (studentScore)가 종료되지 않는다.(내부 옵저버블은 에러를 받으면 여전히 종료된다.)

따라서 아래와 같은 결과를 보여준다.

```swift
--- Example of: materialize and dematerialize ---
next(80)
next(85)
error(anError)
next(100)
```

## 2. Dematerialize

하지만 이런경우 더이상 element가 아닌 event를 다루게 된다. 이를 위해 event를 element로 전환하는 오퍼레이터가 존재하며 아래설명하는 dematarialize이다. 

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/6.%20dematerialize.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/07_Transforming%20Operators/6.%20dematerialize.png?raw=true)

studentScore의 섭스크라이브 부분 코드를 아래와 같이 바꾸자

```swift
studentScore
  // 1
  .filter {
    guard $0.error == nil else {
      print($0.error!)
      return false
    }

    return true
  }
  // 2
  .dematerialize()
  .subscribe(onNext: {
    print($0)
  })
  .disposed(by: disposeBag)
```

filter에서 에러가 아닌것들만 걸러 dematerialize를 통해 element로 변환후 출력된다. error는 error를 출력하고 filter에서 걸러진다. 결과는 아래와 같다.

```swift
--- Example of: materialize and dematerialize ---
80
85
anError
100
```

다시한번 정리하자면 materialize와 dematerialize의 사용으로 내부 옵저버블이 에러로 종료되더라도 외부옵저버블에까지 영향을 끼치지 않고 element를 계속해서 이용할수 있게 되었다!