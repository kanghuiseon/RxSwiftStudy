# Ch8. Transforming Operators in Practice
## Getting started with GitFeed
* RxSwift repository의 가장 최근 활동이 무엇인지 궁금하다! 이번 챕터에서는 최근 활동이 무엇인지 알려주는 프로젝트를 생성할 것이다.
* Github repository의 활동(가장 최신 좋아요, forks, comments)을 보여줄 것이다.
* 프로젝트를 열고 실행하면, Github의 JSON API에서 페치된 가장 최신 활동을 보여줄 table view controller가 보여진다.
<img src= "1" height=200>

* 이 프로젝트는 다음의 두가지를 진행할 것이다.
1. Github의 JSON API에 접근하여, JSON응답을 받고, 그 응답을 collection으로 바꾼다.
2. 이렇게 가져온 collection을 디스크에 저장하고, 서버에서 새로 데이터를 가져오기 전까지의 내용을 table에서 보여준다.  
<br/>
<br/>
<br/>

## Fetching data from the web
* 웹에서 데이터를 가져올때는, URLRequest의 파라미터로 웹 URL을 설정하고, 인터넷으로 보낸다. 그 후, 서버의 응답을 받게 된다.
* GitFeed의 podfile을 잠깐 보자면, RxSwift와 함께 RxCocoa가 있다. RxCocoa는 RxSwift를 기반으로 한 라이브러리이고, 개발에 유용한 API를 구현한다. (Chapter 12, 13에서 자세히 배울 것이다.)
<img src= "2" height=200>

* RxCocoa URLSession extension을 사용해서 GitHub API로부터 JSON을 빠르게 fetch할 것이다.
<br/>

### Using map to build a request
* 우선, Github 서버에 보낼 URLRequest를 생성한다. 
* ActivityController.swift을 열어보자. viewDidLoad에서는 마지막에 refresh()를 호출하는데, 내부에서는 fetchEvents(repo:)를 호출하고, 위에서 설정한 repo 이름인 ReactiveX/RxSwift를 넘겨준다.
* fetchEvents(repo:)에서는, 아래의 코드를 추가한다.
```swift
let response = Observable.from([repo])
```
* web request를 위해, repository의 full name이 필요한데, 이것을 string으로 받는다. URLRequest를 바로 시작하지 않고 stringㅡ올 시작하는 이유는 observable의 input을 좀 더 유연하게 사용하기 위해서이다. 즉, 만약 repository를 변경하더라도 문제가 발생하지 않는다는 의미이다. 
```swift
.map{ urlString -> URL in
    return URL(string: "https://api.github.com/repos/\(urlString)/events")!
}
```
* 그리고 나서, map 연산자를 사용하여, 주소 string을 포함한 URL을 생성한다.
* 꼭 클로저의 리턴타입을 명시해줄 필요는 없지만, map, flatMap을 연달아서 사용하는 경우에는, 컴파일러가 타입을 이해하지 못하는 경우가 있을 수도 있다. 이런 경우에는 오류 표시를 해주므로 그때 고쳐도 상관은 없다.

```swift
.map { url -> URLRequest in
    return URLRequest(url: url)
}
```
* 마지막으로, 생성한 url을 이용해서 URLRequest 형태로 바꾼다.
<img src= "3" height=200>

* 이때까지의 과정을 이미지로 나타내면 위와 같다.
<br/>
<br/>

### Using flatMap to wait for a web response
* flatMap은 보통 transformation chain에 비동기성을 추가하는 것이다. 쉽게 설명하자면, 위의 예시에서, 아래의 사진과 같이 연속적으로  transformation을 진행하면, 이것은 동기적으로 발생한다고 한다.
<img src= "4" height=200>

* 즉, 모든 변형 연산은 서로의 output을 가지고 진행한다.
* 여기에 flatMap연산을 추가하면, 아래의 효과들을 볼 수 있다.
1. 요소들과 complete을 즉시 방출하는 observables를 하나의 observable로 합칠 수 있다.
2. 비동기적으로 일을 하는 observables를 하나로 합칠 수 있다. 비동기 작업을 수행하고, 완료될때까지 ***기다리고*** , 나머지 작업들은 계속해서 작동하도록 할 수 있다. 

<img src = "5" height = 200>

* 이러한 과정들을 GitFeed에 적용하면 위의 모습과 같다. 
* 웹에 요청을 보내고, 응답이 올때까지 기다린다. 하지만 비동기적으로 작업을 수행하기 때문에 다른 작업에 대해서는 계속해서 진행될 수 있도록 한다.
* 연산 마지막에 아래의 코드를 추가해보자.
```swift
.flatMap { request -> Observable<(reponse: HTTPURLResponse, data: Data)> in
    return URLSession.shared.rx.response(request: request)
}
```
* RxCocoa의 reponse(request: ) 메소드를 사용한다. 이 메소드는 Observable<(reponse: HTTPURLResponse, data: Data)를 리턴하는데, 앱이 웹 서버로부터 response를 받을때마다 complete된다.
* response(request: )는 인터넷 연결이 없거나 할 때 에러를 방출할 수 있는데, flatMap 내부에서 에러를 캐치해야한다. 이것에 대해서는 Chapter 14에 자세히 나와있다. 
* flatMap에서는 web request를 보내고, protocols, delegates 없이도, reponse를 받을 수 있다.
* 마지막으로, web requst에 대해 더 많은 구독을 추가하게 하려면, 다음의 연산자를 추가한다.
* share(replay:, scope: )을 이용해서 observable을 공유하도록 하고, 가장 마지막에 방출된 이벤트를 버퍼에 저장한다.
```swift
.share(replay: 1)
```
<br/>
<br/>

### share() vs. share(replay: 1)
* URLSession.rx.response(request: )는 서버에 요청을 보내고, response를 받는다. 그리고 .next 이벤트를 딱 한번 방출하고 complete된다.
* 여기서, 만약 observable이 종료되고 다시 구독을 하면, 새로운 구독이 생성되고,  서버에 새로운 요청을 다시 보낼 것이다. 
* 이러한 상황을 방지하기 위해서, share(replay: scope: )을 이용한다. 이 연산자는 가장 마지막에 replay 숫자만큼 방출된 요소를 버퍼에 저장하고 있다가, 새로운 구독자가 생길 때 바로 제공한다.
* 서버에서 받아온 response를 버퍼에 저장해놨다가, 새로운 구독자가 구독을 시작하면 바로 response를 받을 수 있다.
* scope에는 두 가지 옵션이 있다.
1. .whileConnected : 버퍼에 저장된 response는 더 이상 구독자가 존재하지 않을 때까지만 존재한다. 구독자가 존재하지 않는다면 버려진다. 이 후, 새로운 구독자가 추가되면 새로운 네트워크 response를 얻는다. 
2. .forever : 버퍼로 저장된 response는 영원히 존재한다. 새로운 구독자들을 모두 버퍼에 저장된 response를 얻을 수 있다.
<br/>
* share(replay: scope:)은 complete이 예상되는 sequences나, heavy workload를 발생시키면서, 여러번 구독될 sequence에 사용한다. 

<br/>
<br/>
<br/>

## Transforming the response
* reponse를 받은 이후의 작업에 대해서 알아보자!
* URLSession class가 Data 오브젝트를 줬는데, 바로 사용할 수 없을 때에는 어떻게 할까? 이것을 ***안전하게*** 사용할 수 있어야하는데...
* 다음의 코드를 보자
```swift
response
    .filter { response, _ in
        return 200..<300 ~= response.statusCode
}
```
* filter 연산자를 가지고, 바로 사용할 수 없는 response는 걸러내도록 하자. 오직 200~300사이의 코드만 받을 수 있다. (성공을 의미하는 코드)
* 만약 HTTP response code list를 보고싶다면, [https://bit.ly/1fFATGL](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) 에 들어가서 보자.
* 위의 코드에서 ~=는 잘 알려지지 않은 Swift 연산자인데, 왼쪽의 범위를 사용할 경우, 해당범위에 오른쪽의 값이 포함되어 있는지를 확인한다.
* 이렇게 받은 데이터는 보통 JSON으로 변형된 서버 response일 것이다. 이것을 Decodable 형태의 구조를 사용해서, 받은 data response를 decode 해야 한다. (decodable : json to array or struct etc...)
* Event.swift를 열어서, Event struct를 보자. 이 struct는 이미 Codable protocol을 채택하고 있다. 여기서 Codable을 채택하면, json파일로 변환할 수 있게 된다.
```swift
struct Event: Codable {
  let action: String
  let repo: Repo
  let actor: Actor

  enum CodingKeys: String, CodingKey {
    case action = "type"
    case repo
    case actor
  }
}
```
* 다시 ActivityController.swift로 돌아가서, filter아래에, 새로운 연산자를 추가해준다. 이 곳에서, API response에서 받은 Data를 변형하여, Events list로 만든다.
```swift
.compactMap { _, data -> [Event]? in
    return try? JSONDecoder().decode([Event].self, from: data)
}
```
* 여기서 compactMap은 non-nil 값들은 ***통과*** 시키고 nil이면 필터링한다. 
* 위의 코드를 보자.
    * response object는 버리고, response data만 얻는다.
    * JSONDecoder를 생성하고, response data를 Events 배열로 디코딩한다.
    * try? 를 사용하여, JSON data를 디코딩하는 동안에 decoder가 오류를 발생시키면 nil값을 반환한다.
<img src = "6" height= 200>
    
* transforming 마지막 단계에서는 UI를 업데이트한다. response 마지막에 아래의 코드를 추가하자
```swift
.subscribe(onNext: { [weak self] newEvents in
    self?.processEvents(newEvents)
})
.disposed(by: bag)
```
<br/>
<br/>

### Processing the response
* 여기까지, string으로 시작해서, web request를 생성하고, GitHub에 보내고, 답을 받았다.
* response를 JSON으로 바꾸고, Swift object로 변경했다. 이제는 사용자에게 화면으로 보여줄 차례이다.
* ActivityController에는 이미 processEvents(_:)가 있지만 내부에는 아무것도 없다.
* 이 메소드에서는 repository의 마지막 50개의 이벤트를 받고 subject property인 events에 저장할 것이다.
```swift
private let events = BehaviorRelay<[Event]>(value: [])
...

func processEvents(_ newEvents: [Event]) {
    var updatedEvents = newEvents + events.value
    if updatedEvents.count > 50 {
        updatedEvents = [Event](updatedEvents.prefix(upTo: 50))
    }
    events.accept(updatedEvents)
}
```
* events.accept를 사용해서 list에 새롭게 페치된 이벤트를 추가한다.
* 또한 list를 50개로 제한하여, 테이블 뷰에는 최신 활동만 표시되도록 한다.
* 이미 data source code는 포함되어있으므로, 코드 마지막에 다음의 코드만 추가한다.
```swift
tableView.reloadData()
```
<br/>
<br/>

* 모두 실행하면 아래의 결과를 얻을 수 있다.
<img src = "7" height = 200>

* 하지만, 실행하고나서 다음의 crash를 볼 수 있다.
<img src = "8" height = 200>

* UI 관련된 부분은 무조건 main thread에서 실행이 되어야 하므로, 아래의 코드로 감싸주자.
```swift
DispatchQueue.main.async{
    self.tableView.reloadData()
}
```
<img src = "9" height = 200>

* 추가로 테이블뷰를 refresh하면 절대 사라지지 않는 오류가 발생하는데, 페치가 끝나고나서 이 부분을 없애려면, 아래의 코드를 self.tableView.reloadData() 바로 밑에 추가해준다.
```swift
self.refreshControl?.endRefreshing()
```
* endRefreshing() 은 refresh control을 숨겨줄 것이다.
<br/>
<br/>
<br/>
<br/>

## Persisting objects to disk
* 지금부터는 가장 최근에 페치된 에빈트들을 디스크에 저장시켜놨다가 사용자가 앱을 열면 즉시 볼 수 있도록 할 것이다.
<img src = "10" height = 200>

* 저장할 이벤트의 양이 적기 때문에, 이 이벤트들을 .plist 파일에 저장할 것이다.
* 첫번째로, ActivityController class에 새로운 프로퍼티를 생성한다.
```swift
private var eventFileURL: URL {
  return cachedFileURL("events.json")
}
```
* eventFileURL에는 이벤트를 저장할 디스크 내 파일의 url을 저장한다.
<br/>

```swift
func cachedFileURL(_ fileName: String) -> URL {
    return FileManager.default
        .urls(for: .cachesDirectory, in: .allDomainsMask)
        .first!
        .appendingPathComponent(fileName)
}
```
* cachedFileURL 메소드에서는, 읽고 쓸 수 있는 파일들의 위치에 대한 url을 가져옵니다.
* 이 다음에는, processEvents(_: ) 내부에 아래의 코드를 추가한다.
```swift
let encoder = JSONEncoder()
if let eventsData = try? encoder.encode(updatedEvents) {
    try? eventsData.write(to: eventsFileURL, options: .atomicWrite)
}
```
* 새로 추가한 이벤트 요소를 encode를 이용해 json형태로 생성하고 (eventsData), 생성한 데이터를 write을 이용해, 위에서 설정한 eventsFileURL의 위치에 씁니다.
<br/>
<br/>

* 이제 디스크에 저장된 코드를 읽어와보자! 
* 이 부분에 대한 코드는 viewDidLoad()에 작성해서, 이벤트가 저장되어있는지 확인하고, 만약 있다면, 그것들을 events 에 저장한다.
* viewDidLoad() 로 가서 다음의 코드를 refresh() 위에 작성하자.
```swift
let decoder = JSONDecoder()
if let eventsData = try? Data(contentsOf: eventsFileURL), 
    let persistedEvents = try? decoder.decode([Event].self, from: eventsData){
        events.accept(persistedEvents)
}
```
* 우선, 해당 url에 파일이 존재하면, 디스크에서 Data를 읽어서, JSONDecoder의 decode를 이용해, [Event] 형태로 변형해서, events로 이벤트를 전달합니다.
<br/>
<br/>
<br/>
<br/>

## Add a last-modified header to the request
* 현재 GitFeed는 이전에 페치하지 않은 이벤트만 요청한다. 이렇게 하면 repository에 fork가 없거나, like이 없다면, 서버로부터 빈 응답을 받을 것이고, 네트워크 트래픽을 절약할 것이다. 
* 첫번째로, ActivityController에 파일 이름을 저장할 프로퍼티를 생성한다.
```swift
private var modifiedFileURL: URL {
  return cachedFileURL("modified.txt")
}
```
* Mon, 30 May 2017 04:30:00 GMT. 와 같은 한줄 string 은 .plist file이 필요가 없다.
* 이것은 Last-Modified 이름의 헤더 값으로 서버가 JSON 응답과 같이 보낸다.
* 서버에 다음 요청을 보낼때, 저 헤더를 그대로 서버에게 보내서, 마지막으로 가져온 이벤트와 그 이후에 새롭게 페치한 이벤트가 있는지를 서버가 확인한다.
<img src = "11" height = 200>

* Last-Modified 헤더를 추적하기 위한 subject를 ActivityController에 추가한다.
```swift
private let lastModified = BehaviorRelay<String?>(value: nil)
```
<br/>

* 그리고나서, viewDidLoad( )에서 refresh()위에 다음의 코드를 추가한다.
```swift
if let lastModifiedString = try? String(contentsOf: modifiedFileURL, encoding: .utf8){
    lastModified.accept(lastModifiedString)
}
```
* 이전에 Last-Modified 헤더의 값이 파일에 저장되어있었다면, 다시 lastModifiedString 에 저장한다.
<br/>

* error response를 필터링해보자. fetchEvents()로 가서, 두번째 구독을 생성하고, 추가한다.
* 다음의 코드를 추가한다.
```swift
response
    .filter { response, _ in
        return 200..<400 ~= response.statusCode
    }
```
<br/>
<br/>

* 다음으로 할 일은, 
1. Last-Modified header를 포함하지 않는 모든 response들을 필터링한다.
2. 헤더의 값을 가진다.
3. sequence를 한번 더 필터링하고, 헤더의 값을 고려한다.
<br/>

* 첫번째로, flatMap을 이용해서, Last-Modified header를 가지지 않는 responses를 필터링한다.
* 다음의 코드를 operator chain의 마지막에 추가한다.
```swift
.flatMap { response, _ -> Observable<String> in
    guard let value = response.allHeaderFields["Last-Modified"] as? String else { return Observable.empty() }
    return Observable.just(value)
}
```
* guard 를 이용해서, response가 HTTP 헤더를 포함하는지(Last-Modified이름의) 체크하고, 그 값을 String으로 타입 캐스팅한다.
<img src = "12" height = 200>

<br/>
<br/>

* 원하는 헤더 값을 받으면, lastModified 프로퍼티를 업데이트하고, 디스크에 값을 저장한다.
```swift
.subscribe(onNext: { [weak self] modifiedHeader in
    guard let self = self else { return }
    
    self.lastModified.accept(modifiedHeader)
    try? modifiedHeader.write(to: self.modifiedFileURL, atomically: true, encoding: .utf8)
    })
    .disposed(by: bag)
```
* 구독 내에서, lastModified.value를 최신으로 업데이트하고, 디스크에 저장하기 위해, write를 호출한다.

<br/>
<br/>

* 이제 Github의 API에 요청할때, 저장된 헤더 값을 사용해보자.!
* fetchEvents(repo: )에 가서, url을 요청한 map 연산자를 다음의 코드로 바꾼다.
```swift
.map { [weak self] url -> URLRequest in 
    var request = URLRequest(url: url)
    if let modifiedHeader = self?.lastModified.value {
            request.addValue(modifiedHeader, forHTTPHeaderField: "Last-Modified")
    }
    return request
}
```
* URLRequest를 보내기 전에, lastModified에 값이 있으면, JSON을 가져온 후에 그 값을 Last-Modified 헤더를 request에 추가한다.
* 이 헤더는 GitHub에게 이 헤더 date보다 오래된거에는 관심이 없다고 말해주는거라고 생각하면 된다.
* 이 작업은 네트워크 트래픽을 줄여줄 뿐만 아니라, 데이터를 보내주지 않는 response에 대해서, GitHub API의 사용 제한 수를 증가하지 않는다.
<img src = "13" height = 200>

## Challenge
### Challenge: Fetch top repos and spice up the feed
* 여기서는 가장 최상단에 있는 Swift repositories를 app에서 보여줄 것이다.
<img src="14" height=200>

* 시작하기 위해서, fetchEvents(repo:) 의 let response = Observable.from([repo])를 아래의 코드로 바꾼다.
```swift
let response = Observable.from(["https://api.github.com/search/repositories?q=language:swift&per_page=5"])
```
* 여기서는 top five 의 Swift repositories을 리턴한다.
* GitHub은 각 repositorydml score를 가지고 결과값을 반환한다.
* URL을 URLRequest로 바뀌는 것 까지는 똑같지만, Last-Modified 헤더는 더 이상 필요없다.
* 따라서, raw data 대신에 JSON을 바로 반환해주는 메소드인 URLSession.shared.rx.json(request: )를 사용한다.
* 여기서, [String:Any] 형태의 JSON response를 받아서 items key를 받는다.
* items는 인기 repositories를 보여주는 [String: Any] 형태이다. 그래서 이것들에 대한 full name이 필요하다.
* 이 repo 이름은 user name과 repo name을 포함한다. (icanzilb/EasyAnimation, realm/realm-cocoa, ReactiveX/RxSwift 같이!)
```swift
let response = Observable.from(["https://api.github.com/search/repositories?q=language:swift&per_page=5"])
  .map{ urlString -> URL in
    return URL(string: urlString)!
  }
  .flatMap { url -> Observable<Any> in
    let request = URLRequest(url: url)
    return URLSession.shared.rx.json(request: request)
  }
  .flatMap { response -> Observable<String> in
    guard let response = response as? [String:Any],
          let items = response["items"] as? [[String:Any]] else { return Observable.empty() }
    return Observable.from(items.map { $0["full_name"] as! String})
  }
  .map { [weak self] urlString -> URL in
    return URL(string: "https://api.github.com/repos/\(urlString)/events")!
  }
  .map { [weak self] url -> URLRequest in
    var request = URLRequest(url: url)
    if let modifiedHeader = self?.lastModified.value {
      request.addValue(modifiedHeader,
        forHTTPHeaderField: "Last-Modified")
    }
    return request
  }
  .flatMap { request -> Observable<(response: HTTPURLResponse, data: Data)> in
    return URLSession.shared.rx.response(request: request)
  }
  .share(replay: 1)
```
