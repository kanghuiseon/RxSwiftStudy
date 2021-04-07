# Ch.6 Filtering Operators in Practice

> 이번 챕터에서는 4챕터에서 한 프로젝트에 여러 filtering operator를 사용해보고 어떤 operator가 동일한 효과를 내는지, 똑같은 효과를 내는 다른 filtering operator는 무엇이 있는지 결과적으로 이를 이용해 기존 프로젝트의 문제점을 개선해 봅니다.

# 연산자의 의미 : Operator

- 연산자들은 `Observable<E>` 클래스상의 간단한 메소드들이며, 이 중 몇몇은 `Observable<E>`가 채택하는 `ObservableType` 프로토콜에 정의되어있다.
- 연산자들은 `Observable` 클래스 요소들을 조작하고 결과로써 새로운 observable sequence를 만들어낸다. 이는 아주 편리한데 그 이유는, 여러개의 연산자들을 **연결chain**하여 sequence 내에서 여러가지 작동을 할 수 있기 때문이다.

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/1.%20operators.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/1.%20operators.png?raw=true)

- Observable에 정의되어있는 operator를 통해 기존 Observable 시퀀스에 operation을 행한후 새로운 observable 시퀀스를 얻을수 있다. 여기에 RxSwift의 functional한 측면, 체이닝을 통해 observalbe에 여러 operator를 체인시켜 원하는 시퀀스를 뽑아낼수있다!

# Combinestagram 개선하기

## A. Photo Sequence 다듬기

현재 프로젝트에서는 photoSeqence에 대해 main 뷰컨의 viewDidLoad에서 구독해 images와 UI를 업데이트 한다. 하지만 단순히 이런 동작외에도 사용자는 더많은 기능을 추가하고 싶을수도 있다. 이럴때 마다 구독을 새로 추가해 새로운 Observable Sequence를 생성해야할까? 

### 1. 구독을 공유하기: Share()

- `subscribe(...)`을 하나의 observable에 여러번 호출했을 때 문제가 있을까?
- 앞서 설명했듯이 observable은 lazy하고, pull 구동하는 sequence다. 따라서 사실 `Observable`에 여러개의 연산자를 호출하는 것은 실제 아무 일도 발생시키지 않는다. `subscribe(...)`을 호출한 순간이 `Observable`을 깨워서 요소들을 만들기 시작하도록 하는 것이다. 따라서 observable은 자신이 구독될 때마다 `create` 클로저를 호출한다.
- 아래 코드를 playground에서 구동해보자.

    ```swift
    let numbers = Observable<Int>.create { observer in
    	let start = getStartNumber()
    	observer.onNext(start)
    	observer.onNext(start+1)
    	observer.onNext(start+2)
    	observer.onCompleted()
    	return Disposables.create()
    }

    ```

    - 이 코드를 통해 `srart`, `start+1`, `start+2` 라는 세 개의 숫자를 만들어내는 `Observable<Int>` sequence가 생성되었다.
- 이제 `getStartNumber()`의 구조를 살펴보자.

    ```swift
    var start = 0
    func getStartNumber() -> Int {
    	start += 1
    	return start
    }

    ```

    - 이 함수는 변수를 증가시키고 반환한다. 특별히 잘못된 것이 없어보이는데 정말 그럴까? `number`를 구독하고 살펴보자.

    ```swift
    numbers
    	.subscribe(onNext: { el in
    		print("element [\\(el)]")
    	}, onCompleted: {
    		print("------------")
    })

    /* prints:
     element [1]
     element [2]
     element [3]
     ------------
    */

    ```

- 예상대로 잘 찍히는 것을 알 수 있다. 그렇다면 상기 구독을 복붙해서 한번더 실행해보자. 이 때 결과값은 달라진다.

    ```swift
    /* prints:
     element [1]
     element [2]
     element [3]
     ------------
     element [2]
     element [3]
     element [4]
     ------------
    */

    ```

- 문제는 `subscribe(...)`을 호출할 때마다 구독을 위한 새로운 `Observable`이 생성된다는 것이다. 그리고 각각의 복사본이 이전 결과와 같다는 것을 보장하지 않는다. 심지어 같은 요소의 sequence를 생성한하더라도 굳이 중복의 seqence가 생성되는건 비효율적이다!
- 이러한 불필요한 행위를 방지하고 구독을 공유하기 위해서 `share()` 연산자를 사용한다. Rx 코드의 일반적인 패턴은 하나의 소스 `observable`의 각 결과 값에서 나오는 요소들을 필터링해 여러 개의 sequence들을 생성하는 것이다.
- Combinestagram을 이용하여 `share()`를 사용해보자.
- **MainViewController.swift**의 `actionAdd()` 함수내의 `photosViewController.selectedPhotos`를 아래의 코드로 바꾸자.

    ```swift
    let newPhotos = photosViewController.selectedPhotos
        .share()

    newPhotos

    ```

- 아래 그림과 같이 observable에 대해 각각 구독하는 대신에,

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/3.%20previous.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/3.%20previous.png?raw=true)

- 아래 그림 처럼 하나의 `Observable`을 `share()`하여 여러 번 구독할 수 있다.

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/4.%20share.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/4.%20share.png?raw=true)

- 다음 과정으로 넘어가기전에 `share`에 대해서 좀 더 알아보자.
- `share`는 구독자의 수가 0에서 1로 될 때만 구독을 생성한다. 두 번째, 세 번째 그리고 구독자들이 sequence를 관찰하기 시작할 때, `share`는 이미 생성한 구독을 사용하여 추가 구독자들과 공유한다. 만약 공유된 sequence에 대한 모든 구독이 dispose 되면 (예를 들어, 더 이상의 구독자가 없는 경우), `share`는 *공유된 sequence 또한 dispose 한다*. 다른 가입자가 다시 관찰을 시작하면 `share`는 *새로운* 구독을 생성한다.
    - **참고**: `share()`는 구독이 영향을 받기 전까지는 어떠한 값 방출도 내지 않는다. 반면에 `share(replay:scope:)`은 마지막 몇개의 방출 값에 대한 버퍼를 가지며 새로운 관찰자가 구독했을 때 이를 제공해준다.
- sharing 연산자는 complete 되지 않는 observable에 사용하기에 안전하다.  또는 complete 후에 새로운 구독이 수행되지 않는다고 보장할 수 있는 경우에 사용하기 좋다. Ch.8 "Transforming Operations in Practice."에서 이에 대해 자세히 배울 수 있다.??

### 2. 모든 요소 무시하기: igonoreElements()

- 모든 요소들을 필터하는 `ignoreElements()` 연산자를 다뤄보자. 정확히 말하면 observable에서 방출하는 모든 요소, `.next` 이벤트를 무시하고 `.complete` 나 `.error` 이벤트만 통과시킨다.
    - ingoreElemnets는 사실 일반적인 observable에서 .next 이벤트를 필터링해 `completable` observable로 변환하는 오퍼레이터라고 생각하면 된다.
- 해당 연산자를 사용해보기위해서 combinestagram에 새 기능을 추가한다. addAction으로 사진이 추가되고 메인 뷰컨으로 돌아왔을때 네비게이션바의 좌측 상단에 조그만 프리뷰 이미지를 표시하도록 하자.
- 이기능을 추가할경우 사진이 추가되는 매순간, .next 이벤트마다 반응할필요가 없이 .complete 이벤트시에만 UI를 업데이트하면 되기때문에 newPhotos에대해 ignoreElements를 통해 이기능을 구현해보자.
- `MainViewController`내부에 `updateNavigationIcon()` 함수를 추가하고 `actionAdd()`의 마지막 부분에 아래 코드를 추가하자.

    ```swift
    newPhotos
          .ignoreElements()
          .subscribe(onCompleted: { [weak self] in
              self?.updateNavigationIcon()
          })
          .disposed(by: bag)
    // ...

    private func updateNavigationIcon() {
      let icon = imagePreview.image?
        .scaled(CGSize(width: 22, height: 22))
        .withRenderingMode(.alwaysOriginal)
      navigationItem.leftBarButtonItem = UIBarButtonItem(image:
    icon,
        style: .done, target: nil, action: nil)
    }
    ```

    - 이제 addAction을 통해 사진을 추가하고 메인뷰컨으로 돌아올때 .complete이벤트에 의해 화면 좌상단에 작은 미리보기 이미지가 추가되는것을 확인할수 있다.

    ![https://user-images.githubusercontent.com/28700930/113820707-08a7f880-97b6-11eb-9273-5ac122861668.png](https://user-images.githubusercontent.com/28700930/113820707-08a7f880-97b6-11eb-9273-5ac122861668.png)

### 3. 필요 없는 요소들 필터링하기: filter(_: )

- 모든 요소들을 필터링하는 `ignoreElements()` 외에 몇 가지의 요소만 필터링하고 싶을 때 사용할 수 있는 `filter(_:)` 연산자가 있다. 예를 들어 사진이 세로보기 형태라면 조합했을 때 잘 맞지 않을 것이다. 따라서 세로보기 사진은 필터링해보자.
- `actionAdd()`의 `newPhotos`에 대한 첫 번째 구독을 대해 다음과 같이 `filter`를 삽입하자.

    ```swift
    .filter { newImage in
        return newImage.size.width > newImage.size.height
    }

    ```

    - 이제 `newPhotos`가 방출하는 각각의 사진들은 구독되기 전에 `filter`를 거치게 된다. 여기서는 사이즈를 확인해서 가로길이가 세로길이보다 긴 사진만 통과하도록 했다.

### 4. 기본적인 uniqueness filter 구현하기: filter(_: )

- 지금의 Combinestagram에선 똑같은 사진을 여러번 추가하여 콜라주를 만들 수 있지만, 지금부터는 같은 사진을 여러번 추가하는 것을 필터링하여 서로 다른 사진만을 조합하게 하고 싶다. filter를 이용해 unique한 사진만을 콜라주하도록 구성해보자.
- addAction에서 새로추가되는 이미지들에 대해 uniqueness를 보장하기위한 가장 간단한 좋은 방법은 이미지 데이터를 해시하거나, asset URL을 해시해서 저장, 비교하는 방법이 있지만 이번 예제에서는 imageCashe라는 이미지들의 byte를 해싱하는 구조체를 하나 선언해 image가 추가될때마다 이 해시에 값이 있는지 확인하고 없을경우 새로운 이미지를 추가하고 있을경우에는 동일한 이미지라 판단해 filter해본다.
    - 더나은방법은 chapter 9 에서 scan operator를 이용해 구현해본다.
- `MainViewController` 클래스에 다음과 같은 새로운 프로퍼티를 추가하자.

    ```swift
    private var imageCache = [Int]()
    ```

    - 그후 image uniqueness를 위한 `filter`를 아까 가로세로길이를 비교했던 filter밑에 추가해준다.

    ```swift
    .filter { [weak self] newImage in
    	let len = UIImagePNGRepresentation(newImage)?.count ?? 0
    	guard self?.imageCache.contains(len) == false else { return false }
    	self?.imageCache.append(len)
    	return true
    }

    ```

    - 이 코드를 통해 새로운 이미지의 PNG 데이터의 byte 수를 상수 `len`으로 저장할 수 있다. 만약 `imageCache`가 같은 값을 이미 가지고 있다면 중복 이미지라 판단하고 `false`를 반환할 것이다.
- 새로 구현한 기능이 잘돌아가도록   `actionClear()`에 이미지캐시를 비우는 작업을 추가해준다.

    ```swift
    imageCache = []
    ```

### 5. 조건에 부합하는 동안 요소들 취하기: takeWhile(_)

- Combinestagram의 가장 큰 버그는 사용자가 6개의 사진을 추가했을 때 main view controller의 + 버튼이 비활성화 되지만 phto view controller에서는 6개 이상의 사진을 추가할수 있는점이다.
- 이 문제를 addAction에서 6개 보다 많은 사진이 추가되지 않도록 takeWhile 오퍼레이터를 이용해 해결해보자. `takeWhile(_)` 연산자는 모든 요소들이 확실한 조건에 부합했을 때만 필터링하도록 설정할 수 있다. Boolean 조건에 따라 `takeWhile(_)`은 조건이 `false`일 때의 모든 요소를 무시한다.
- `actionAdd()`로 돌아가서 `newPhotos`의 첫 번째 구독 부분 바로 아래에 다음 코드를 추가하자.

    ```swift
    newPhotos
        .takeWhile { [weak self] image in
            let count = self?.images.value.count ?? 0
            return count < 6
         }
    ```

    - `takeWhile(...)`은 이미지의 총 개수가 6보다 작을 동안만 이미지를 통과시킨다.
    - 만약 `self`가 `nil`일 때는 기본값 `0`을 줄 수 있도록 `??` 연산자를 사용한다.
    - weak self는 아직도 모르겠다..
    - **NOTE:** 이번예제에서는 메인뷰컨의 property에 직접적으로 접근해 구성했지만 리액티브 프로그래밍에서 이런방식은 선호되지 않는다. chapter9에서 상태를 저장하기위해 viewController를 사용하지않고 observable을 combine하는 기법에 대해 배운다.
- 이제 photoViewController에서 사진을 6장이상 선택해도 이내용이 메인뷰컨에 반영되지 않는다.

## B. 사진 선택 개선하기

- **PhotosViewController.swift**에서 새로운 `Observable`을 생성하고 새로운 방법으로 필터하여 화면의 UX를 개선해 본다.

### 1. PHPhotoLibrary authority observable: 새로운 권한 observable 만들기

- 현재 컴바인스타그램의 첫 구동을 생각해보면 문제가 있다. addAction을 실행하면 사진권한을 요청하고, 허용할경우에 photoViewController에서 바로 사진이 뜨지않는다. mainVIewController로 다시 돌아갔다가 addAction을 다시 눌러야 사진이 로드되는걸 확인할수 있다.
- 현재로서는 photoViewController에서 권한을 확인하고 사진을 reload할 방법이 존재하지 않아서 그렇다. 그렇다면 사진권한에 대한 정보를 bool값으로 가지는 새로운 observable을 생성하고 이를 구독해 이 문제를 해결해보자!
- **PHPhotoLibrary+rx.swift** 라는 새로운 이름의 파일을 생성하고 다음 코드를 입력하자.

    ```swift
    import Foundation
    import Photos
    import RxSwift

    extension PHPhotoLibrary {
       static var authorized: Observable<Bool> {
           return Observable.create { observer in
               return Disposables.create()
           }
       }
    }
    ```

    - 사용자가 접근을 허가 했는지 여부에 따라 두가지 방향으로 진행될 수 있다.

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/6.%20Where.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/6.%20Where.png?raw=true)

- 플로우차트를 `return Disposables.create()` 상단에 코드로 표현해보자.

    ```swift
    DispatchQueue.main.async {
      if authorizationStatus() == .authorized {
        observer.onNext(true)
        observer.onCompleted()
      } else {
        observer.onNext(false)
        requestAuthorization { newStatus in
          observer.onNext(newStatus == .authorized)
          observer.onCompleted()
    	  }
    	} 
    }

    ```

    - 사진권한을 옵저버블로 구성해봤다. 권한이 이미 있다면 true를 emit후에 .completed 없다면 false를 emit후 권한을 재요청하고 그결과를 emit한다. 그후에 마찬가지로 .completed를 emit한다.
    - DispatchQueue.main.async로 구성한이유는 해당 observable이 현재 쓰레드를 블락하는 경우가 생겨서는 안되기 떄문이다.

### 2. 접근 허용되었을 때 사진들 리로드하기: 권한 observable 구독하기

- 우리가 구성한 권한 플로우차트에 따라 사진 라이브러리 접근에 성공하는경우는 두가지 시나리오가 존재한다.
    - 하나는 alert가 떴을 때, 사용자가 **OK** 버튼을 누르는 것

        ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/7.%20first.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/7.%20first.png?raw=true)

    - 다른 하나는 이미 허용한 다음에 접근한 경우

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/8.%20second.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/8.%20second.png?raw=true)

- 즉 사진을 리로드하는경우는 권한접근이 성공한는 경우, `PHPhtoLibrary.authorized` 에서 true가 마지막 element로 방출되는경우이다. 그렇기에 마지막 element인 true를 받을경우 사진을 reload할수 있도록 구독을 추가하자!
- 아래 코드를 **PhotosViewController.swift**의 `viewDidLoad()`에 추가하자.

    ```swift
    let authorized = PHPhotoLibrary.authorized
        .share()

    authorized
        .skipWhile { $0 == false }
        .take(1)
        .subscribe(onNext: { [weak self] _ in
            self?.photos = PhotosViewController.loadPhotos()
            DispatchQueue.main.async {
                self?.collectionView?.reloadData()
            }
        })
        .disposed(by: bag)

    ```

    - `skipWhile(_:)`: `true`가 나오기 전까지는 `false`들을 무시한다.
    - `take(1)` `true`가 필터링되어 나오면, 하나의 요소를 받은 뒤 그 뒤에 나오는 아이들은 무시한다. (우리가 정한 로직에서는 당연히 true가 항상 마지막 요소이기 때문에 take(1)을 쓸필요는 없지만 명시적으로 take(1)을 사용해 의도를 정확하게 표현하고 권한에 대한 로직이 변경된다하더라도 이를통해 첫 true요소에서 사진을 리로드하고 이후 내용은 무시할수 있다!
- `subscribe(...)` 클로저 내부에서는 콜렉션뷰를 리로드 하기 전에 메인 쓰레드로 전환한다. 왜 이런 작업을 해야할까? 이를 위해서는 subscribe내의 코드가 실행되는 시점에 대해서 살펴볼필요가 있다. 즉 권한 요청시에 true가 결과로 전달되는 subscribe가 실행되기 직전의 상황으로 돌아가보자.

    ```swift
    requestAuthorization { newStatus
    	observer.onNext(newStatus == .authorized)
    	observer.copleted()
    }
    ```

    - reqestAuthorization 후에 completion 클로저에서 obsevable로 결과 emelent를(true인경우)를 전달한다. 하지만 이 내부 클로저는 어떤 쓰레드에서 실행되는지 알수 없기때문에 subscirbe가 되는 시점에 명시적으로 mainThread에서 UI가 업데이트 될수 있도록 하는게 중요하다.
    - 만일 UI가 백그라운드에서 업데이트되게된다면 UIKit이 크래시 될것이다.
    - **NOTE:** 사실 RXSwift에서는 이런 GCD를 이용해 쓰레들 전환하지 않고 schedular를 이용해서 이를 관리하는데 이는 chapter 15에서 자세하게 다룬다.
    - UI업데이트는 백그라운드에서 일어나면 안되나?

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/9.%20UI%20thread.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/9.%20UI%20thread.png?raw=true)

### 3. 사용자가 접근을 허용하지 않았을 때 에러메시지 표시하기: 권한 observable 구독하기

- 지금까지는 사용자가 권한을 허용한경우에 대해서만 구독을 처리해봤다. 그렇다면 사용자가 권한요청을 거부하는경우에 구독처리는 어떻게 이루어져야할까?
- 권한요청이 거절되는 경우에대한 로직을 살펴보자.
    - 최초 1회 앱 실행시 사용자가 접근을 허용하지 않았을 때

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/10.%20third.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/10.%20third.png?raw=true)

    - 이후에도 계속 접근은 허용되지 않을 것이다.

    ![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/10.%20third.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/10.%20third.png?raw=true)

- 두 가지 경우 모두 flowchart에서 오른쪽경우를 따라 권한을 요청하고 false를 받기 때문에 sequence 요소는 같다. 중요한건 두번째 false가 이 observable 시퀀스의 결과라는점과 마지막 요소인 false가 우리가 핸들링 해야할 요소라는점이다.(둘다 같은말이다)
- 이제 새로운 구독을 추가해 권한이 거절될경우에 에러메세지를 alert하는 구독을 추가해보자.
- 이 내용을 `viewDidLoad()`에 추가하자

    ```swift
    authorized
      .skip(1)
      .takeLast(1)
      .filter { !$0 }
      .subscribe(onNext: { [weak self] _ in
        guard let errorMessage = self?.errorMessage else { return }
        DispatchQueue.main.async(execute: errorMessage)
      })
      .disposed(by: bag)

    ```

    - skip(1)을 통해 첫 엘리먼트를 걸렀고 takeLast(1)을 통해 마찬가지로 마지막 요소만을 받을수 있도록했다. (중복된 filter를 넣는건 비효율적으로 보이지만 이런 명시적인 중복코드의 이점에 대해서 아래에서 설명하도록 하겠다.)
    - **참고**: `filter {$0 == false}`는 `filter {!$0}`로 축약될 수 있고 이는 `filter{!}`로 축약될 수 있다.
- `errorMessage`를 구현해야 하므로 `PhotoViewController` 내부에 다음 코드를 추가한다.

    ```swift
    private func errorMessage() {
      alert(title: "No access to Camera Roll",
        text: "You can grant access to Combinestagram from the Settings app")
        .asObservable()
        .subscribe(onCompleted: { [weak self] in
          self?.dismiss(animated: true, completion: nil)
          _ = self?.navigationController?.popViewController(animated: true)
        })
        .disposed(by: bag)
    }
    ```

    - 에러메세지의 동작원리에 대해서는 chapter4의 challange 1을 참고하자

---

**같은 효과를 주는 filter operator들과 명시적인 filter 중복 사용의 이유**

- `skip`, `takeLast`, `filter`를 함께 사용해서 원하는 결과를 얻을 수 있는 코드를 작성했다. 하지만 좀 더 리팩토링이 가능할 것 같다. `PHPhotosLibrary.authorized`에서 다음 부분을 생각해보자.

    ```swift
    authorized
    	.skip(1)
    	.filter { $0 == false }

    ```

    - 구조 파악을 통해 여기엔 최대 2개 이상의 요소가 항상 있다는 것을 알 수 있다. 따라서 첫 번째 값을 skip하고 뒤따라 나오는 요소를 필터할 수 있다.

    ```swift
    authorized
    	.takeLast(1)
    	.filter { $0 == false}

    ```

    - 마지막 요소 이전 것은 모두 무시하고 마지막 요소가 `false`인지만 확인하는 방법도 있다.

    ```swift
    authorized
    	.distinctUntilChanged()
    	.takeLast(1)
    	.filter { $0 == false }

    ```

    - `skip`과 `takeLast`를 `distinctUntilChanged()`로 대체할 수도 있다.
- 상기 코드들은 모두 같은 효과를 낸다. 따라서 구독에 대한 코드를 조금 더 줄이는 것이 가능하다. 하지만 이 것은 sequence 로직이 *절대* 변하지 않을 것임을 *확신*할 때만 가능하다. 만약 다음 iOS 버전이 출시된다면 어떻게 될까? `grant-access-alert-box` 로직이 변경되지 않는다고 확신할 수 있을까?
- 따라서 `skip`, `takeLast`, `filter` 조합을 유지하는 것이 여기서는 최선의 방법이다.

## C. 시간 기반 필터 연산자 사용하기

- 시간 기반 연산자에 대한 자세한 내용은 Ch.11 "Time Based Operators"에서 배울 것이다. 하지만 몇 가지 연산자들은 필터링 연산자이기도 해 이들을 다뤄본다.
- 시간 기반 연산자는 **Scheduler**를 사용한다. 이는 중요한 개념으로, 지금 다룰 예제에서는 `MainScheduler.instance`를 사용할 것이다.

### 1. 주어진 시간 간격 뒤에 구독 완료시키기: take(_: )

- 현재는 사진접근 권한이 없을경우 alert를 띄우고 close를 누르면 mainViewController로 돌아가는 방식이다. 시간 기반 필터 연산자를 사용해서 alert가 뜨고 5초뒤에 자동으로 close를 눌렀을때와 같은 반응이 일어나도록, 즉 구독이 .complete 되도록 구성해보자!
- **PhotoViewController.swift**를 열고 `errorMessage()` 메소드를 확인하자. `alert(title:..., discription: ... )` 뒤에 다음과 같은 코드를 작성하자.

    ```swift
    .asObservable()
    .take(5.0, scheduler: MainScheduler.instance)

    ```

    - `take(_:scheduler:)`는 `take(1)`이나 `takeWhile(...)` 같은 필터링 연산자이다. `take(_:scheduler:)`는 주어진 시간동안 소스 sequence로부터 엘리먼트를 받다가 주어진 시간이 지나면 결과 sequence를 `.complete` 시킨다.
    - 다시 프로젝트를 확인해보면 5초후에 자동으로 구독이 완료되어 close를 누르지 않더라도 메인뷰컨으로 돌아가는 모습을 볼수있다.
    - **NOTE:** take 오퍼레이터는 `Completable` 타입에 사용이 불가능하기때문에 `.asObservable()` 메소드를 통해 Observable 타입으로 변환시켜준다.

### 2. throttle을 사용해서 부하가 많은 구독에 대한 작업 줄이기: throtle(_:scheduler: )

- sequece의 현재 요소에만 관심이 있고 이전 값들은 필요없는 경우가 있다.
- 현재 combinestagram의 addAction에서 사진을 선택할때를 생각해보자 메인뷰컨에선 사진이 선택될때마다 콜라주를 만들어 imagePreview를 업데이트 한다. 새로운 사진을 받는 이상, 이전의 콜라주는 쓸모 없다. 빠르게 여러개의 사진을 성공적으로 탭했더라도 구독은 새로운 콜라주를 일일히 만들 것이다. 사실 리소스 낭비에 가깝다. 이런 상황은 현업에서 생각보다 빈번하게 일어난다.
- 이런 상황을 해결하기위해, 특정 정해진 시간동안 반복해서 일어나는 엘리먼트를 무시하는 throtle을 사용할수 있다.
- 예제를 자세히 보기 위해 **MainViewController.swift**의 `viewDidLoad()`를 살펴보자.

    ```swift
    images
        .subscribe(onNext: { [weak imagePreview] photos in
            guard let preview = imagePreview else { return }
            
            preview.image = photos.collage(size: preview.frame.size)
        })
        .disposed(by: bag)

    ```

- `images.asObservable()` 바로 다음에 다음 코드를 삽입하자.

    ```swift
    .throttle(0.5, scheduler: MainScheduler.instance)

    ```

- `throttle(_:scheduler:)`는 주어진 시간 내에 뒤따라오는 요소들을 필터한다. 따라서 어떤 사용자가 사진 하나를 선택하고 바로 다음 사진을 선택하는데 0.2초가 걸렸다면 `throttle`은 첫번째 요소를 필터하고 두번째 사진만 내뱉을 것이다. 이렇게하면 첫 번째 중간 콜라주를 작성하는 시간이 절약된다.
- 당연히 `throttle`는 한가지 이상의 요소들과 함께 작업할 수 있다. 만약 사용자가 5개의 사진을 선택간격 0.5초 이내로 빠르게 탭하였다면, `throttle`은 첫 4개를 필터하고 5번째 요소만 내뱉을 것이다.

![https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/12.%20throttle.png?raw=true](https://github.com/fimuxd/RxSwift/raw/master/Lectures/06_Filtering%20Operators%20in%20Practice/12.%20throttle.png?raw=true)

- `throttle`을 사용할 수 있는 상황은 다음과 같다.
    - 현재의 텍스트를 서버 API에 보내는 검색 텍스트 필드를 구독할 때, 사용자가 빠르게 타이핑 한다는 가정이 있다면 타이핑을 완전히 마쳤을 때의 텍스트를 서버에 보내도록 `throttole`을 이용할 수 있다.
    - 사용자가 bar 버튼을 눌러 view controller를 present modal 할 때, present modal이 여러번 되지 않도록 더블/트리플 탭을 방지할 수 있다.
    - 사용자가 손가락을 이용해 화면을 드래그할 때, 드래그가 끝나는 지점에만 관심이 있을 수 있다. 드래그 중일 때의 터치 위치는 무시하고 터치 위치가 변경을 멈추었을 때의 요소만 고려할 수 있다.
- `throttle(_:scheduler:)`는 너무 많은 입력값을 받고 있을 때 아주 유용하게 쓸 수 있다.

# Challenge

## 1. viewDidLoad()의 구독 공유하기

```swift
let currentImages = images.share()
  
currentImages
  .throttle(.milliseconds(500), scheduler: MainScheduler.instance)
  .subscribe(onNext: { [weak imagePreview] photos in
    guard let preview = imagePreview else { return }
    preview.image = photos.collage(size: preview.frame.size)
  })
  .disposed(by: bag)

currentImages
  .subscribe(onNext: { [weak self] photos in
    self?.updateUI(photos: photos)
  })
  .disposed(by: bag)
```

image에 share오퍼레이터를 추가해서 share할수있다.

## 2. actionClear()에서 프리뷰이미지 제거하기

```swift
updateNavigationIcon()
```

updateNavigation()을 추가해서 프리뷰이미지를 clear버튼을 누를때 같이 제거할수 있다.

---

### Artwork/images/designs: from RxSwift: Reactive Programming in Swift book, available at [http://www.raywenderlich.com](http://www.raywenderlich.com/)