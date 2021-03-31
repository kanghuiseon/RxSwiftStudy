# Ch4. Observables and Subjects in Practice
## Using a subject/relay in a view controller

* MainViewController.swift파일을 열고, 다음의 코드를 추가한다.
```swift
private let bag = DisposeBag()
private let images = BehaviorRelay<[UIImage]>(value: [])
```
* 다른 class에서 사용하지 않기 때문에 두 변수 모두 private으로 선언한다. (Encapsulation, 캡슐화)
* relay이 같은 경우에는 next 이벤트만을 보내기 때문에 보통 ui와 관련해서 사용을 많이 한다.

<img src = "1" height = 200>

* 그림에서 보는 것과 같이, dispose bag은 view controller에서 관리한다. 
* view controller가 release되자마자, 모든 observable의 구독들이 취소된다.
* 이 때문에 따로 메모리를 관리하지 않아도 돼서 편리하다. 그냥 bag 하나 만들어서 그곳에 버리기만 하면 알아서 수거가 된다.
* 하지만 root vew controller는 앱이 종료되기 전에 release되기 때문에, 여기서는 위와 같은 상황은 벌어지지 않는다.
<br/>
<br/>

* starter 프로젝트를 처음 연 상태에서는 +버튼을 탭할때마다 똑같은 사진이 images에 추가된다.
* 다음의 코드를 actionAdd() 에 추가시킨다.
```swift
let newImages = images.value + [UIImage(named: "IMG_1907.jpg")!]
images.accept(newImages)
```
* 우선 가장 최근 이미지 배열을 가지고 있는 images.value에 지정한 이미지 배열을 합치고, images relay가 구독자에게 이벤트를 방출하도록 한다.
* 여기서 Relay는 onNext가 아닌 accept를 사용한다.
* 가장 처음의 images.value는 빈 배열이고, + 버튼을 탭할때마다 images는 새로운 이미지를 추가하여 .next이벤트를 방출한다.
<br/>
<br/>


```swift
images.accept([])
```
* 만약 clear 버튼을 눌렀을때, 모든 이미지를 사라지게 하고 싶다면, actionClear() 에 위의 코드를 추가하여, 빈 배열을 가진 이벤트를 구독자에게 방출하도록 한다.
<br/>
<br/>

### Adding photos to the collage
* 구독자를 추가하기 위해서, viewDidLoad()에 다음의 코드를 추가한다. relay도 ObservableType을 채택하기 때문에, 바로 구독을 추가할 수 있다.
```swift
images
    .subscribe(onNext: { [weak imagePreview] photos in 
        guard let preview = imagePreview else { return }
        
        preview.image = photos.collage(size: preview.frame.size)
    })
    .disposed(by: bag)
```
* images에 이미지가 추가될때마다 구독자는 UIImage 배열에 대해서, 콜라쥬를 진행하고 preview화면에 보이도록 설정한다.
* 이번 챕터에서는 viewDidLoad에 구독을 추가하였지만, 다른 클래스에서도 이에 대한 구독을 할 수 있도록 한다.

<br/>
<br/>

### Driving a complex view controller UI
* UI는 다음과 같이 개선할 수 있다.
1. 만약 사진이 한장도 없거나, 사용자가 clear버튼을 누른 직후에, clear 버튼이 작동하지 않도록 한다. 
2. 동일하게 사진이 한장도 없다면, save버튼도 작동하지 않도록 한다.
3. 만약 사진이 홀수개로 존재한다면, save버튼이 작동하지 않도록 한다.
4. 사진의 개수를 제한하는것도 좋은 방법이다.
5. 현재 사진의 개수에 따라 view controller title을 변경하는 것도 좋다.
<br/>

* RxSwift에서는 images에 구독을 추가하여, 바로 UI를 업데이트할 수 있다.
```swift
images
    .subscribe(onNext: { [weak self] photos in 
        self?.updateUI(photos: photos)
    })
    .disposed(by: bag)
```
* 위의 코드를 viewDidLoad()에 추가한다.

* 새로운 사진을 추가할때마다, updateUI(photos: )를 호출하여 UI를 업데이트 해준다.
* 이 함수는 다음과 같다.
```swift
private func updateUI(photos: [UIImage]){
    buttonSave.isEnabled = photos.count > 0 && photos.count % 2 == 0
    buttonClear.isEnabled = photos.count > 0
    itemAdd.isEnabled = photos.count < 6
    title = photos.count > 0 ? "\(photos.count) photos" : "Collage"
}
```
* 이와 같이 Rx를 사용하면, 간단하게 몇 줄을 추가함으로써 모든 UI를 변경할 수 있게 된다.

<br/>
<br/>

## Talking to other view controllers via subjects
* 이번 챕터에서는 main view controller와 PhotoViewController class를 연결해본다.


```swift
let photosViewController = storyboard!.instantiateViewController(withIdentifier: "PhotosViewController") as! PhotosViewController

navigationController!.pushViewController(photosViewController, animated: true)
```
* 우선 navigation stack에 PhotosViewController을 push하기 위해 MainViewController.swift에서 actionAdd()에 위의 코드를 추가한다. (기존의 코드는 주석처리하거나 제거한다.)
* 앱을 실행하면 다음과 같이 + 버튼을 탭하면 카메라 앨범이 뜨는 것을 볼 수 있다.
<img src = "2" height=200>


* 만약 기존의 방식대로 다음 코드를 진행했다면, main view와 photos view가 서로 통신할 수 있도록 delegate protocol을 추가했을 것이다. 하지만 이것은 Rx 다운 방식이 아니다.!
* RxSwift에서는, Observable을 사용한다.! Observable은 어떤 observer라도 이벤트를 전달할 수 있기 때문에 유용하게 쓰인다.
<br/>
<br/>

### Creating an observable out of the selectd photos
* PhotosViewController에 import RxSwift구문을 추가하고, 사용자가 앨범에서 사진을 클릭할때마다 .next이벤트가 방출되도록 한다.
* PublishSubject를 이용하여 선택된 이미지를 방출하도록 할건데, 다른 class에서 onNext(_) 이벤트를 방출하도록 하면 안되므로 private으로 선언하여 캡슐화한다.
* 다음의 코드를 PhotosViewController에 추가한다.
```swift
private let selectedPhotosSubject = PublishSubject<UIIImage>()
var selectedPhotos: Observable<UIImage> {
    return selectedPhotosSubject.asObservable()
}
```
* private인 Publishsubject는 선택된 사진들을 방출할 것이고, public인 selectedPhotos는 subject의 observable을 방출한다.
* 이 public 프로퍼티를 구독하면, main controller에서 어떠한 방해도 없이 photo sequence를 관찰할 수 있다.
<br/>
<br/>

```swift
if let isThumbnail = info[PHImageResultIsDegradedKey as NSString] as? Bool, !isThumbnail {
    self?.selectedPhotosSubject.onNext(image)
}
```
* Collection view의 cell을 클릭할때마다, 선택된 사진을 방출하는 코드를 추가해보자.
* collectionView(_: didSelectItemAt) 메소드에서는 이미 사용자가 사진을 클릭했을때 flash를 통해 visual feedback을 제공하고 있다.
* imageManager는 PHCachingImageManager의 인스턴스 객체로, 미리보기 썸네일을 검색하거나 생성 할 수 있는 객체이다.
* imageManager.requestImage(...) 에서 파라미터로 image와 info를 얻을 수 있다. closure에서, 이미지가 선택되면, 파라미터로 받은 image 를 가지고 .next 이벤트를 방출한다.

<br/>
<br/>

### Observing the sequence of selected photos
* MainViewController.swift에서 다음의 코드를 추가하여서, 선택된 사진의 sequence를 관찰하도록 한다.
```swift
photosViewController.selectedPhotos
    .subscribe(
        onNext: { [weak self] newImage in
        
        },
        onDisposed: {
            print("Completed photo selection")
        }
    )
    .disposed(by: bag)
```
* selectedPhotos observable을 관찰하여서, .next 이벤트가 방출될때 (사용자가 사진을 선택하면) 원하는 코드를 실행시키도록 한다.
* onNext closure안에 다음의 코드를 추가한다.
```swift
guard let images = self?.images else { return }
images.accept(images.value + [newImage])
```
<img src = "3" height = 200>
* 앱을 실행시키면, 카메라 앨범에서 사진을 선택하면, 메인화면에 이미지가 추가된것을 볼 수 있다.

<br/>
<br/>

### Disposing subscriptions - review
* 위의 코드에서 onDisposed를 통해, 구독이 취소될때마다 print를 하도록 했다. 하지만 앱을 구동시키고나서는 저  메세지를 볼 수 없다. 그 이유는 disposed(by: bag)을 통해 MainViewController가 해제될때 구독도 취소되거나, sequence가 error or complete 이벤트를 방출할때 구독이 끝나기 때문이다.
* 만약 controller가 화면에서 사라질때, observer가 구독을 취소하도록 하려면, 화면에서 사라질때, .completed 이벤트를 방출시키도록 한다.
* PhotosViewController.swift를 열고, subject의 onComplete() 메소드를 viewWillDisappear(_:) 에 추가하도록 한다.
```swift
selectedPhotosSubject.onCompleted()
```

<br/>
<br/>

## Creating a custom observable
* 현재까지는 BehaviorRelay, PublishSubject, Observable을 가지고 코드를 진행하였다.
* 이것과 더불어 custom한 Observable도 만들 수 있다.
* Photos 프레임워크를 이용해서, 생성한 사진 콜라쥬를 저장하도록 하자.
* PhotoWriter라는 이름의 custom class를 생성하고, 사진을 저장하기위한 Observable을 생성한다. 만약 사진이 디스크에 저장이 되면, asset ID와 .completed이벤트를 방출할 것이다. 
<br/>

### Wrapping an existing API
* Classes/PhotoWriter.swift 파일을 열고 import RxSwift 코드를 추가한다.
* 그리고 나서, PhotoWriter에 static method를 하나 생성한다.
```swift
static func save(_ image: UIImage) -> Observable<String> {
    return Observable.create { observer in
        var savedAssetId: String?
        PHPhotoLibrary.shared().performChanges({
            let request = PHAssetChangeRequest.creationRequestForAsset(from: image)
            savedAssetId = request.placeholderForCreatedAsset?.localIdentifier
        }, completionHandler: { success, error in
            DispatchQueue.main.async{
                if success, let id = savedAssetId {
                    observer.onNext(id)
                    observer.onCompleted()
                } else{
                    observer.onError(error ?? Errors.couldNotSavePhoto)
                    
                }
            }
        })
        return Disposables.create()
    }
}
```
* save(_:)는 사진을 저장하고나서, unique identifier를 방출하도록 하는 Observable<String>을 return 한다. 
* Observable.create는 새로운 Observable을 생성하고, 클로저에서 해당 Observable에 대한 코드를 작성한다.
* performChanges(_: completionHandler:) 메소드의 첫번째 클로저에서, photo asset을 생성하고, 두번째 클로저에서는 만약 위의 코드가 성공한다면, assetID를 방출하고, 그렇지 않으면 error 이벤트를 방출한다.
* PHAssetChangeRequest.creationRequestForAsset(from:)을 통해 새로운 photo asset을 생성하고, savedAssetId에서 해당 identifier를 저장한다.
* completionHandler에서는 이과정이 성공하면, .next를 방출하고 .completed 이벤트를 방출한다.


<br/>
<br/>

* 이 과정까지 끝내고 나면, 왜 하나의 .next 이벤트만 방출하는 Observable이 필요한가? 에 대한 궁금증이 생길 수 있다.
* 이전의 챕터들을 생각해봤을때, 다음의 연산들을 통해서도 Observable을 생성할 수 있다.
1. Observable.never(): 아무것도 방출하지 않는 연산자.
2. Observable.just(_:) : 하나의 요소, .completed 이벤트만 방출하는 연산자.
3. Observable.empty() : .completed 이벤트만 방출하는 연산자.
4. Observable.error(_) : .error 이벤트만 방출하고, 어떤 요소도 방출하지 않는 연산자.
* 위와 같이, observable은 아무것도 방출하지 않거나, 그 이상의 .next 이벤트를 방출한다. 그리고 .completed 나 .error 이벤트로 종료시킬 수도 있다.
* PhotoWriter의 경우, 단 하나의 .next 이벤트와 .completed or .error만을 방출시키는데, 이것을 하나의 연산자로 구현할 수는 없을까?

<br/>
<br/>

## RxSwift traits in practice
### Single
* Single은 .success(Value) 이벤트와 .error이벤트만을 방출할 수 있는 특별한 연산자이다.
* 여기서 .success란, .next + .completed를 의미한다.

<img src = "4" height = 200>

* 이러한 특징은, 파일을 저장하거나, 다운로드, 또는 디스크에서 데이터를 가져오거나 값을 만들어내는 비동기적인 것에서 유용하게 사용될 수 있다.
1. PhotoWriter.save(_) 처럼 하나의 요소만을 방출하는 연산을 할 때, single을 사용할 수 있다.
2. single을 이용해서 둘 이상의 요소가 들어오는지를 확인할 수 있다. 만약 둘 이상 들어오면, error를 방출한다.
(observabledp .asSingle()을 추가하면, Single로 변환할 수 있다.)

<br/>
<br/>

### Maybe
* Maybe는 Single과 좀 비슷하지만 한가지 다른 점은 성공을 하더라도 어떤 값도 방출하지 않을 수 있다는 것이다.

<img src = "5" height = 200>

* 만약 사진과 관련된 예시를 생각한다면, 앱은 custome한 사진 앨범에 사진들을 저장하고 있을 것이다. 
* UserDefaults에 앨범의 identifier을 저장하고, 앨범을 열고, 사진을 저장할때마다 해당 id를 사용할 것이다.
* open(albumId: )-> Maybe<String> 연산자를 만들어서 다음의 상황을 다룰 수 있다.
1. 주어진 id가 존재하는 경우에는 .completed 이벤트를 방출한다.
2. 사용자가 앨범을 지운다면, 새로운 앨범을 만들고 새로운 id를 가진 .next 이벤트를 방출하여, UserDefaults를 계속해서 유지할 수 있다.
3. 뭔가 에러가 났을 때는 photos library에 접근할 수 없기 때문에 .error 이벤트를 방출한다.

* Single처럼, Maybe.create({...}) 나, observable에 .asMaybe()를 붙여 사용할 수 있다.

<br/>
<br/>

## Completable
* completable은 .completed 나 .error 이벤트만을 방출한다.
<img src = "6" height = 200>

* ignoreElements() 연산자를 이용하면, observable sequence를 completable로 바꿀 수 있다.
* 이렇게 되면 모든 next이벤트는 무시되고 completed 나 error 이벤트만 방출한다. 

<br/>

* 또한 Completable.create { ... }를 이용해서, completable sequence를 만들 수 있다. 
* completable sequence는 비동기식 연산이 성공했는지 안했는지를 알고 싶을 때 많이 사용이 된다.

<br/>

* 앞에서 했던 Combinestagram을 보자. 사용자가 작업을 하는 동안에 자동 저장되는 기능을 추가하고 싶다.
* 백그라운드 큐에서 문서가 비동기식으로 저장되는것을 추가하고 싶다. 완료가 되면 notification을 띄우고, 실패하면 alert를 화면에 나타내는 방식으로 코드를 추가한다.
```swift
saveDocument()
    .andThen(Observable.from([createMessage))
    .subscribe(onNext: { message in 
        message.display()
    }, onError: { e in
        alert(e.localizedDescription)
    })
```
* andThen 연산자는 더 많은 completable연산자나 observables를 연결하고, 마지막 결과를 구독할 수 있게 한다.
<br/>
<br/>

### Subscribing to your custom observable
* PhotoWriter.save(_) observable은 단 하나의 요소만을 방출하기 때문에 single로 사용하기 좋다. 
* MainViewController.swfit의 actionSave()에 다음의 코드를 추가한다.
```swift
guard let image = imagePreview.image else { return }

PhotoWriter.save(image)
    .asSingle()
    .subscribe(
        onSuccess: { [weak self] id in
            self?.showMessage("Saved with id: \(id)")
            self?.actionClear()
        },
        onError: { [weak self] error in
            self?.showMessage("Error", description: error.localizedDescription)
            }
    ).disposed(by: bag)
}
```
* 현재 콜라쥬를 저장하기 위해 위의 함수를 호출한다. Observable을 single로 변환하고, 메세지가 성공인지 에러인지를 뜨게 하고, 성공을 했다면 clear()를 하도록 한다.
<br/>
<br/>

## Challenges
### Challenge 1: It's only logical to use a Single
* 이미 PhotoWriter.save는 하나의 요소만을 방출하기 때문에 이것의 Return type을 single로 바꿀 수도 있다.
* 리턴타입을 Single<String>으로 바꾸고, Observable.create를 Single.create로 바꿔준다.
* 여기서, 기존의 Observable.create는 파라미터로 observer를 받아서, 여러개의 값을 방출할 수도 있고, 이벤트를 멈출 수도 있다. 
* 하지만, Single.create는 파라미터로 클로저를 받기 때문에, .success(T), .error(E) 둘 중 하나를 방출할 수 있도록 한다.
```swift
static func save(_ image: UIImage) -> Single<String> {
  return Single.create { observer in
    var savedAssetId: String?
    PHPhotoLibrary.shared().performChanges({
      let request = PHAssetChangeRequest.creationRequestForAsset(from: image)
      savedAssetId = request.placeholderForCreatedAsset?.localIdentifier
    }, completionHandler: { success, error in
      DispatchQueue.main.async {
        if success, let id = savedAssetId {
          observer(.success(id))
        } else {
          observer(.error(error ?? Errors.couldNotSavePhoto))
        }
      }
    })
    return Disposables.create()
  }
}
```
<br/>
<br/>

### Challenge 2 : Add custom observable to present alerts
* MainViewController.swift를 열고, showMessage(_:description:) 메소드를 보자.
* 이 메소드는 화면에 alert를 띄우고, 사용자가 alert를 끄려고 Close버튼을 탭할때의 Callback을 제공한다.
* 이 challenge를 완성하기 위해서, 다음과 같이 해보자
1. UIViewController의 extension을 추가하고, 주어진 타이틀과 메세지를 화면에 보여주고 Completable을 리턴하도록 하자.
2. 사용자가 alert를 닫도록하기 위해 Close 버튼을 추가한다.
3. 구독이 취소되었을때도 alert controller를 종료시킨다.
```swift
import UIKit
import RxSwift

extension UIViewController {
    func alert(title: String, text: String?) -> Completable {
        return Completable.create(subscribe: { [weak self] completable in
            let alertVC = UIAlertController(title: title, message: text, preferredStyle: .alert)
            let close = UIAlertAction(title: "Close", style: .default, handler: { _ in
                completable(.completed)
            })
            alertVC.addAction(close)
            self?.present(alertVC, animated: true, completion: nil)
            return Disposables.create {
                self?.dismiss(animated: true, completion: nil)
            }
        })
    }
}
```
