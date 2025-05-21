## 🌀 App Life Cycle

### 📱 App-based Life Cycle (iOS 12 이하 또는 단일 Scene 사용 시)

<img src="https://github.com/user-attachments/assets/64200630-c8c5-40b8-8fd1-a3188fac9fac" width="500"/>

<br />

- Not Running: 앱이 전혀 실행되지 않았거나 종료됨
- Inactive: 앱이 실행 중이지만 이벤트를 수신하지 않음
- Active: 앱이 전면에서 실행 중이고 사용자 이벤트를 받고 있음
- Background: 앱이 백그라운드에서 실행 중
- Suspended: 앱이 백그라운드에 있으며 코드 실행을 중단한 상태
- AppDelegate 중심으로 앱의 전반적인 흐름 관리
- 전체 앱에 해당하는 전역 상태 또는 설정을 여기에 선언

```swift
// AppDelegate.swift
var sharedData = 0 // 전역 데이터

// ViewController.swift
if let appDelegate = UIApplication.shared.delegate as? AppDelegate {
    appDelegate.sharedData
}
```

<br />

### 🪟 Scene-based Life Cycle (iOS 13 이상, 멀티 윈도우 대응)

<img src="https://github.com/user-attachments/assets/0c7ee2bd-0c53-4bcb-8caa-43efd0de3718" width="500"/>

<br />

- Unattached: Scene이 아직 UI와 연결되지 않은 초기 상태
- 앱에 여러 Scene(UIWindowScene)이 존재할 수 있음
- 각 Scene은 SceneDelegate가 관리
- 독립적인 UI/상태관리 가능

```swift
// ViewController.swift
if let sceneDelegate = UIApplication.shared.connectedScenes.first?.delegate as? SceneDelegate {
    sceneDelegate.sharedData
}
```

- SceneDelegate 외부에서 `sceneDidBecomeActive()` 등을 호출하고 싶을 땐
`NotificationCenter`를 통해 옵저빙해야 함

<br />

## ⚙️ UIApplication 관련 상수

- `UIApplication.openSettingsURLString`: 앱 설정 화면으로 이동
- `UIApplication.openNotificationSettingsURLString`: 알림 설정
- `UIApplication.openDefaultApplicationsSettingsURLString`: 기본 앱 설정

<br />

## 🖥️ Screen → View 구조

```swift
// 계층 구조
UIScreen                     // 디바이스 화면
└── UIWindowScene            // 하나의 화면 장면
    └── UIWindow             // 실제 UI를 띄우는 윈도우
        └── UIViewController // 논리적 UI 단위 (화면 1개)
            └── UIView       // 구체적인 UI 구성 요소
```

> SwiftUI에서는 `WindowGroup`, `NavigationStack`, `View`로 계층 단순화
> 

<br />

## 🎬 ViewController Life Cycle

<img src="https://github.com/user-attachments/assets/2b6fa383-1461-47e5-8637-16585a7515b7" width="500"/>

<br />

- `viewDidLoad()`: 루트 뷰가 메모리에 생성되고 나서 VC 레벨의 초기화
- `viewWillAppear()`: 루트 뷰가 계층에 추가되기 직전
- `viewIsAppearing()`: 루트 뷰가 계층에 추가된 직후, 뷰의 배치를 업데이트
- `viewDidAppear()`: 루트 뷰가 계층에 추가되고 나서
- `viewWillDisappear()`: 루트 뷰가 계층에서 사라지기 직전
- `viewDidDisappear()`: 루트 뷰가 계층에서 사라지고 나서

> `view.window?.windowScene?.delegate`는 뷰가 화면에 표시된 후에만 유효
→ `viewDidAppear()` 사용 권장
> 

<br />

## 🧩 View Controller Presentation

### Card Modal (SwiftUI - `.sheet`)

- Presenting VC(화면을 띄운 쪽)는 `viewWillDisappear`와 `viewDidDisappear`가 호출되지 않음

| Card Modal 열기 | Presenting VC | Presented VC |
| --- | --- | --- |
| 1 |  | viewDidLoad |
| 2 |  | viewWillAppear |
| 3 |  | viewIsAppearing |
| 4 |  | viewDidAppear |

| Card Modal 닫기 | Presenting VC | Presented VC |
| --- | --- | --- |
| 1 |  | viewWillDisappear |
| 2 |  | viewDidDisappear |

<br />

## Full Screen (SwiftUI - `.fullScreenCover`)

- Presenting VC는 완전히 사라짐 → `viewWillDisappear`와 `viewDidDisappear`가 호출됨

| Full Screen 열기 | Presenting VC | Presented VC |
| --- | --- | --- |
| 1 |  | viewDidLoad |
| 2 | viewWillDisappear |  |
| 3 |  | viewWillAppear |
| 4 |  | viewIsAppearing |
| 5 |  | viewDidAppear |
| 6 | viewDidDisappear |  |

| Full Screen 닫기 | Presenting VC | Presented VC |
| --- | --- | --- |
| 1 |  | viewWillDisappear |
| 2 | viewWillAppear |  |
| 3 | viewIsAppearing |  |
| 4 | viewDidAppear |  |
| 5 |  | viewDidDisappear |

<br />

## 🧱 View

### 역할

- 데이터 출력 (label, image 등)
- 이벤트 처리 (tap, swipe 등)
- 뷰 계층 관리 (subview/superview)

<br />

### 계층

- `superview`: 현재 뷰를 포함하는 부모 뷰
- `subviews`: 현재 뷰가 포함하고 있는 자식 뷰 목록

<br />

### View Tagging

- 뷰에 `tag` 값을 설정하여 식별 가능

```swift
view.viewWithTag(100)
```

<br />

### UIView.animate()

- UIView의 속성을 애니메이션으로 변경

```swift
UIView.animate(withDuration: 0.3) {
    view.viewWithTag(100).backgroundColor = .yellow
}
```

<br />

## 🤔 느낀점

- 특정 Life cycle에 로직 구현할 때 `super.` 까먹지 말자!

<br />

## ✅ TODO

- [UIViewController 공식문서](https://developer.apple.com/documentation/uikit/uiviewcontroller) 읽어보기
