## 🧰 Interface Builder

- Xcode에서 UI를 시각적으로 구성할 수 있는 도구
- UI 요소 배치, 연결(Outlet, Action), 설정 등을 시각적으로 처리 가능

<br />

## 🎬 Scene

- 하나의 UIViewController와 그에 연결된 View를 말함
- 하나의 화면을 나타내며, `Storyboard` 내에서 각각의 Scene은 하나의 ViewController로 구성됨

<br />

## 🖼️ Canvas

- Interface Builder에서 UI를 구성하는 영역
- View와 제어 요소(버튼, 레이블 등)를 배치하는 공간

<br />

## 🧾 Document Outline

- 현재 Scene에 포함된 모든 UI 요소의 계층 구조를 나열한 목록
- 계층 구조를 명확하게 파악하고 관리하기 용이

<br />

## 📚 Library (⇧ + ⌘ + L)

- 버튼, 레이블 등 UIKit 요소를 검색하고 드래그하여 Canvas에 배치 가능
- 코드 없이 UI 요소를 추가할 수 있음

<br />

## 🧵 Connection Well

- 스토리보드에서 드래그 앤 드롭으로 Outlet, Action 등을 연결할 때 사용하는 작은 원형 포인트
- 오른쪽 패널 또는 코드로 드래그하여 연결 가능

![image](https://github.com/user-attachments/assets/17f16b21-bb28-4da5-a39c-30b86c97db69)

<br />

## 🔗 Outlet / Outlet Collection

- IBOutlet: 스토리보드 UI 요소를 코드에서 참조할 수 있도록 연결
- IBOutlet Collection: 같은 타입의 여러 IBOutlet을 배열 형태로 묶어 제어

```swift
@IBOutlet weak var label: UILabel!
@IBOutlet var labels: [UILabel]!
```

<br />

## 🧠 Action

- UI 요소의 이벤트(예: 버튼 클릭)에 반응하도록 연결된 함수

```swift
@IBAction func buttonTapped(_ sender: UIButton) {
    print("버튼이 눌렸습니다.")
}
```

<br />

## ⚠️ UIAlertController() & UIAlertAction()

- 사용자에게 메시지를 띄우고 입력/선택을 받을 때 사용

```swift
let alert = UIAlertController(title: "알림", message: "계속하시겠습니까?", preferredStyle: .alert)
let confirmAction = UIAlertAction(title: "확인", style: .default)
let cancelAction = UIAlertAction(title: "취소", style: .cancel)

alert.addAction(confirmAction)
alert.addAction(cancelAction)

present(alert, animated: true)
```

<br />

## ⌨️ .becomeFirstResponder() / .resignFirstResponder()

- UIResponder 객체에게 포커스를 주거나 포커스를 해제하는 메서드

```swift
@IBOutlet weak var idField: UITextField!

idField.becomeFirstResponder() // 키보드 포커스 요청
idField.resignFirstResponder() // 포커스 해제 및 키보드 내리기
```

<br />

## 🔄 viewWillTransition()

- 화면 회전과 같은 디바이스 방향 전환이 일어날 때 호출되는 메서드

```swift
override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
    super.viewWillTransition(to: size, with: coordinator)
    print("화면 방향이 바뀝니다.")
}
```

<br />

## 📋 UITableViewDataSource

- TableView의 데이터를 공급하는 프로토콜

```swift
extension ViewController: UITableViewDataSource {
	func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
	    return items.count
	}
	
	func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
	    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
	    
	    cell.textLabel?.text = items[indexPath.row]
	    
	    return cell
	}
}
```

<br />

## 🧭 Delegate Pattern (대리자 패턴)

### 💡 개념

- Delegate Pattern은 한 객체가 자신이 해야 할 일을 다른 객체에 위임(delegate)하는 디자인 패턴
- 역할을 나누고, 재사용성과 유연성을 높이는 데 유리
- UIKit에서는 많은 컴포넌트가 이 패턴을 사용
    
    (예: `UITableViewDelegate`, `UITextFieldDelegate` 등)
    

<br />

### 🧩 구성 요소

| 요소 | 설명 |
| --- | --- |
| Protocol (Delegate 정의) | 위임받을 역할(함수들)을 선언 |
| Delegate Object | 실제로 작업을 수행할 객체 |
| Delegator (위임자) | Delegate의 함수를 호출하는 주체. 보통 `weak` 참조 |

<br />

### 📱 UIKit에서의 활용 예

- UITextFieldDelegate
    
    ```swift
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return true
    }
    ```
    
<br />

- UITableViewDelegate / UITableViewDataSource
    
    ```swift
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("셀 선택됨")
    }
    ```
    
<br />

### 🔁 UIKit Delegate 패턴 vs SwiftUI 선언형

- UIKit
    
    ```swift
    class ViewController: UIViewController {
    	let fruits = ["Apple", "Banana", "Orange"]
    	let languages = ["Swift", "C#", "Java", "HTML", "Go"]
    }
    
    extension ViewController: UITableViewDataSource {
    	// 섹션 수 정의 (과일, 언어)
    	func numberOfSections(in tableView: UITableView) -> Int {
    		return 2
    	}
    
    	// 각 섹션의 셀 수
    	func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    		return section == 0 ? fruits.count : languages.count
    	}
    
    	// 각 셀에 표시할 데이터 설정
    	func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    		// 셀 재사용 큐에서 꺼내오기
    		let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
    		
    			// 섹션에 따라 표시할 데이터 선택
    			cell.textLabel?.text = indexPath.section == 0
    				? fruits[indexPath.row]
    				: languages[indexPath.row]
    				
    			return cell
    	}
    }
    
    // UITableView에서 발생하는 이벤트 처리
    extension ViewController: UITableViewDelegate {
    	// 셀 선택 시 호출되는 메서드
    	func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    		let selected = indexPath.section == 0
    			? fruits[indexPath.row]
    			: languages[indexPath.row]
    		print(selected)
    	}
    }
    ```
    
    > `UITableViewDelegate`와 `UITableViewDataSource`를 채택하고 구현
    > 
    >  섹션, 셀 수, 셀 클릭 이벤트를 delegate 메서드로 처리함

<br />

- SwiftUI
    
    ```swift
    import SwiftUI
    
    struct ContentView: View {
    	let fruits = ["Apple", "Banana", "Orange"]
    	let languages = ["Swift", "C#", "Java", "HTML", "Go"]
    
    	var body: some View {
    		// List는 UITableView에 해당
    		List {
    			Section(header: Text("Fruits")) {
    				ForEach(fruits, id: \.self) { fruit in
    					Text(fruit)
    						.onTapGesture { print(fruit) }
    				}
    			}
    
    			Section(header: Text("Languages")) {
    				ForEach(languages, id: \.self) { language in
    					Text(language)
    						.onTapGesture { print(language) }
    				}
    			}
    		}
    		.listStyle(.insetGrouped)
    	}
    }
    ```
    
    > `List`와 `Section`, `ForEach`를 사용해 구조적으로 뷰를 구성
    > 
    > 별도 delegate/protocol 없이도 `onTapGesture`로 이벤트 처리
