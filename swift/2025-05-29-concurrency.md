## NotificationCenter로 메모 업데이트 전달

- 메모 수정시 `NotificationCenter.default.post`로 전송
- 수신 측에서는 .`addObserver`를 통해 수신 후 UI 갱신

```swift
// 전송
NotificationCenter.default.post(name: .memoDidUpdate, object: nil, userInfo: ["memo": entity])

// 수신
NotificationCenter.default.addObserver(forName: .memoDidUpdate, object: nil, queue: .main) { [weak self] noti in
    if let memo = noti.userInfo?["memo"] as? MemoEntity { ... }
}
```

<br />

## NotificationCenter와 메모리 누수 방지

- 클로저 캡처 시 `self`를 약한 참조로 선언 (`[weak self]`)
- 옵저버를 변수에 저장 후, `deinit`에서 `removeObserver()` 호출

```swift
var token: NSObjectProtocol?

override func viewDidLoad() {
		...
    token = NotificationCenter.default.addObserver(forName: .memoDidUpdate, object: nil, queue: .main) { [weak self] _ in
		    ...
    }
}

deinit {
    if let token {
        NotificationCenter.default.removeObserver(token)
    }
}
```

- 참고문서
    - [https://seizze.github.io/2019/11/28/iOS의-Notification-Center-확장,-그리고-Retain-Cycle-피하기.html](https://seizze.github.io/2019/11/28/iOS%EC%9D%98-Notification-Center-%ED%99%95%EC%9E%A5,-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Retain-Cycle-%ED%94%BC%ED%95%98%EA%B8%B0.html)

<br />

## Swipe로 메모 삭제

```swift
extension ListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            DataManager.shared.delete(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .automatic)
        }
    }
}
```

### 테이블 뷰를 업데이트하고 데이터 삭제시, `Crash`가 발생하는 이유

- `UITableView`는 내부적으로 `dataSource`의 상태와 일치해야 함
- 즉, `deleteRows(at:)`를 호출할 때, 해당 indexPath에 해당하는 데이터가 `dataSource`에 존재해야 함

<br />

## Tab Button으로 메모 삭제

```swift
@IBAction func deleteMemo(_ sender: Any) {
    if let memo {
        DataManager.shared.delete(entity: memo)
        navigationController?.popViewController(animated: true)
				DataManager.shared.fetch()
    }
}
```

### `navigationController`가 Optional인 이유

- 현재 뷰 컨트롤러가 `UINavigationController`에 임베드되어 있을 때만 값이 존재

<br />

## KeyPath

- Key(문자열) + Path(경로)
- `value(forKey:)` 문자열 기반 접근

```swift
// Key-value Coding(문자열 키를 사용해서 속성에 접근)을 지원하는 클래스
class Person: NSObject {
    @objc let name: String = "Jane Doe"
    @objc var age: Int = 20
}

let p = Person()

print(p.name) // Jane Doe
print(p.value(forKey: "name")) // Optional(Jane Doe)

print(p.value(forKey: "asdf")) // Crash!!
// this class is not key value coding-compliant for the key asdf
```

- 문자열에 오타가 발생하면 `Crash` 발생

<br />

### Keypath string expr

```swift
let ketPath = #keyPath(Person.name) // Compile Time Check로 보다 안전

print(p.value(forKeyPath: keyPath)) // Optional(Jane Doe)
```

- `.value()`의 리턴 타입이 `Any?`이므로, 타입 캐스팅 필요

<br />

### Keypath expr + Keypath Type

- 클래스/구조체 모두 지원
- KVC 구현 필요 X

```swift
struct Person2 {
		let name: String = "Jane Doe"
    var age: Int = 20
}

let p2 = Person2()
print(p2[keyPath: \Person2.name]) // Jane Doe
print(p2[keyPath: \Person2.age])  // 20
```

<br />

## FetchedResults

- CoreData에서 TableView나 CollectionView와 같은 UI 컴포넌트와의
효율적인 데이터 연동을 위해 사용하는 객체

<br />

### `NSFetchedResultsController`

- CoreData의 데이터 변경사항을 자동 감지하고, UI와 동기화할 수 있도록 돕는 컨트롤러

<br />

### 핵심 구성

```swift
let mainContext = NSPersistentContainer(name: "Memo0528").viewContext
let request = MemoEntity.fetchRequest()
let sortByDateDesc = NSSortDescriptor(
		keyPath: \MemoEntity.insertDate,
		ascending: false
)
request.sortDescriptors = [sortByDateDesc]

fetchedResults = NSFetchedResultsController(
    fetchRequest: request,
    managedObjectContext: mainContext,
    sectionNameKeyPath: nil,
    cacheName: nil
)
```

- `fetchRequest`: 어떤 데이터를 어떻게 가져올지 정의
- `managedObjectContext`: 데이터를 fetch할 컨텍스트 (여기선 `mainContext`)

<br />

### 초기 fetch

```swift
do {
		try fetchedResults.performFetch()
} catch {
		print(error)
}
```

- 앱 시작 시, 데이터를 처음 가져오는 데 사용
- 이후에는 변경 사항을 자동 감지

<br />

### 데이터 접근

```swift
let memo = DataManager.shared.fetchedResults.object(at: indexPath)
```

- `object(at:)`으로 메모 데이터를 직접 가져옴

<br />

### 데이터 변경에 따른 자동 갱신

```swift
func controllerWillChangeContent(...) {
    tableView.beginUpdates()
}

func controller(_:didChange:at:for:newIndexPath:) {
    switch type {
    case .insert: tableView.insertRows(at:)
    case .delete: tableView.deleteRows(at:)
    case .update: tableView.reloadRows(at:)
    case .move:   tableView.moveRow(at:to:)
    }
}

func controllerDidChangeContent(...) {
    tableView.endUpdates()
}
```

- `NSFetchedResultsControllerDelegate`를 구현하여 CoreData의 변화를 UI에 자동 반영

<br />

## Concurrency (동시성)

- GCD, Swift Concurrency, OperationQueue
- 프로세스: 실행 중인 앱 인스턴스, 독립된 메모리 공간 보유
- 스레드: 프로세스 내부 실행 흐름
- IPC (Inter-Process Communication): 서로 다른 프로세스 간 데이터 공유 방법

|  | 멀티프로세스 | 멀티스레드 |
| --- | --- | --- |
| 컨텍스트 스위칭 | 느림, 오버헤드 큼 | 빠름, 오버헤드 작음 |
| 데이터/리소스 공유 | IPC 구현 | 공유 메모리/리소스에 바로 접근 가능 |
| 안전성 | 높음 | 낮음 |

<br />

### Synchronous(동기) vs Asynchronous(비동기)

|  | 동기 | 비동기 |
| --- | --- | --- |
| 특징 | 작업이 끝날 때까지 기다림 | 작업이 끝날 때까지 기다리지 않음 |
| 장점 | 실행 흐름 파악 쉬움 | 병렬 처리로 빠른 실행 가능 |
| 단점 | 전체 수행 시간 증가 | 구현 복잡, 디버깅 어려움 |

<br />

### Serial vs Parallel vs Concurrent

|  | Serial (순차 방식) | Parallel (병렬 방식) | Concurrent (동시처리 방식) |
| --- | --- | --- | --- |
| 실행 순서 | 순차적으로 한 번에 하나씩 | 동시 시작 O, 동시 실행 O | 동시 시작 X, 동시 실행 O |
| 특징 | 이해하기 쉽고, 작업들 간 영향 X → 동기화 필요 X | 빠르지만 동기화 필요, 멀티 코어/ 멀티 프로세스 환경에 적합 | 빠르지만 동기화 필요, 싱글 프로세스/멀티 프로세스 환경에서 구현 가능 |

<br />

### 스레드 종류

- MainThread: UI 업데이트, 사용자 이벤트 처리
- Worker Thread, Background Thread: 무거운 작업

<br />

## GCD (Grand Central Dispatch)

- Apple이 제공하는 스레드 관리 API
- DispatchQueue: 스레드 관리 객체, 작업을 담는 큐
- Work Item: 큐에 담기는 실행할 코드 블록

<br />

### 주요 DispatchQueue 종류

```swift
let serialQueue = DispatchQueue(label: "serial") // 순차 실행
let concurrentQueue = DispatchQueue(label: "concurrent", attributes: .concurrent) // 병렬 실행
let mainQueue = DispatchQueue.main // 메인 스레드
let globalQueue = DispatchQueue.global() // 시스템 제공 전역 큐
```

<br />

### 주요 메서드

```swift
// 1. 동기 실행 (작업이 끝날 때까지 대기)
serialQueue.sync {
    print("Sync 작업")
}

// 2. 비동기 실행 (작업 완료 기다리지 않음)
concurrentQueue.async {
    print("Async 작업")
}

// 3. 지연 실행
serialQueue.asyncAfter(deadline: .now() + 5) {
    print("5초 후 실행")
}

// 4. 반복 병렬 처리
DispatchQueue.concurrentPerform(iterations: 10) { i in
    print("index: \(i)")
}
```

<br />

---

<br />

## 🤔 Comment

- FetchedResults를 이용하니까 코드가 훨씬 깔끔해진 것 같아서 CoreData를 구현할 때 써봐야겠다.
- SwiftUI에서 ToastAlert를 구현할 때 DispatchQueue를 사용했었는데 UI 외적으로는 어떤 상황에 쓰이는지 궁금하다.
