## 🧱 Container View Controller vs Content View Controller

- Container View Controller
    - 다른 View Controller들을 자식으로 포함(Embed)하는 VC
    - 예: `UINavigationController`, `UITabBarController`, `UISplitViewController`
    - 자식 뷰컨트롤러의 life cycle을 관리하고 화면 전환 흐름을 제어
- Content View Controller
    - 실질적인 화면의 콘텐츠를 담당하는 일반적인 VC

<br />

## 🎬 Segue
- storyboard 기반 화면 전환 방식
- 종류: Show (push), Modal, Popover, Custom
- Identifier를 통해 어떤 segue인지 구분하고 동작 커스터마이징 가능

```swift
let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
```

<br />

## 🔁 `prepare(for: , sender: )`

- Segue로 화면 전환 전에 목표 ViewController에 데이터를 전달할 때 사용

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let vc = segue.destination.children.first as? ComposeViewController {
        vc.listVC = self
    }
}
```

> SwiftUI의 `NavigationLink(destination:)`의 초기화 인자 전달과 유사한 역할
> 

<br />

## ❗ Implicitly Unwrapped Optional (`!`)

- 옵셔널이지만, 초기화 후에는 반드시 값이 있다고 보장할 수 있을 때 사용
- 보통 IBOutlet처럼 storyboard와 연결되는 경우 사용됨

```swift
@IBOutlet weak var toDoTableView: UITableView!
```

<br />

## 🔗 Coupling (결합도)

- 두 객체가 얼마나 강하게 연결되어 있는지를 나타내는 개념
- UIKit에서 delegate는 coupling을 낮추기 위한 수단
- 반면, 강한 의존성이 있으면 재사용과 테스트가 어려워짐

> SwiftUI는 바인딩, 클로저 전달 방식으로 더 낮은 결합도를 유도
> 

<br />

## 🚀 App Life Cycle (앱 생명 주기)

1. `main()`
    - 앱 실행의 시작점 (`@main` attribute 사용됨)
2. `UIApplicationMain()`
    - 앱 인스턴스 생성 및 이벤트 루프 시작
3. Main UI Load
    - Info.plist에서 지정한 storyboard 파일 실행
4. Start Initialization
    - AppDelegate → SceneDelegate → 초기화 로직 실행
5. Restore UI State
    - 상태 복원(이전 실행 상태, 사용자 위치 등)
6. End Initialization
    - 초기 화면이 사용자에게 보임

<br />

## 📀 UIKit에서의 화면 간 데이터 전달 방식

### 1. 직접 참조 방식

```swift
// ListViewController.swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let vc = segue.destination.children.first as? ComposeViewController {
        vc.listVC = self
    }
}

// ComposeViewController.swift
var listVC: ListViewController?
listVC?.toDoList.append(text)
listVC?.toDoTableView.reloadData()
```

- 한쪽 뷰 컨트롤러를 다른 쪽에서 직접 참조함
- 코드량이 적고 구현이 직관적이나, 강한 결합

<br />

### 2. Delegate 패턴

```swift
// ListViewController.swift
extension ListViewController: ToDoDelegate {
    func composeViewController(_ vc: UIViewController, didInsert text: String) {
        toDoList.append(text)
        toDoListView.reloadData()
    }
}

// ComposeViewController.swift
weak var delegate: ToDoDelegate?
delegate?.composeViewController(self, didInsert: text)
```

- `ToDoDelegate` 프로토콜을 만들어 위임 처리
- Delegate를 통해 약한 결합 유지

<br />

### 3. NotificationCenter

```swift
// ListViewController.swift
token = NotificationCenter.default.addObserver(
    forName: .toDoDidInsert, object: nil, queue: .main
) { [weak self] noti in
    if let todo = noti.userInfo?[Key.todo] as? String {
        self?.toDoList.append(todo)
        self?.toDoTableView.reloadData()
    }
}

// ComposeViewController.swift
NotificationCenter.default.post(
    name: .toDoDidInsert,
    object: nil,
    userInfo: [Key.todo: text]
)
```

- 옵저버 패턴을 통해 양방향 독립성 보장
- 다수의 객체와 통신할 때 유리

<br />

### 🧾 UIKit 방식 vs SwiftUI 대응 비교

| UIKit 방식 | SwiftUI 대응 | 결합도 | 상태 공유 | 재사용성 |
| --- | --- | --- | --- | --- |
| 직접 참조 (`listVC`) | `@Binding` | 중간 | 부모-자식 | 중간 |
| Delegate 패턴 | `ObservableObject` | 낮음 | 양방향 | 높음 |
| NotificationCenter | `@EnvironmentObject` | 매우 낮음 | 전역 | 높음 |

<br />

## 🤔 느낀 점

- UIKit 의 Delegate Pattern 을 보고 나니,<br />
처음 보는 코드의 흐름이 SwiftUI보다 한눈에 파악되기 어렵다고 느꼈다.
- Delegate Pattern 이<br />
“상위 Content VC” 에서 로직을 구현하고<br />
“하위 Content VC” 가 갖다 쓰는 개념으로 이해하면 되려나?<br />
Vue.js 의 `Provide`/`Inject` 같이 주입시키는 것과 비슷해 보인다.
