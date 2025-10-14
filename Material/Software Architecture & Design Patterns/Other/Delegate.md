## Delegate Pattern
**The Delegate Pattern** is a Behavioral Design Pattern where one object delegates responsibility for a specific task to another object.
In iOS, it's used to customize behavior or respon to events without subclassing.
>A delegate is like saying: "Hey, if something happens, I'll call you"

```
Delegator (Sender) → Delegate (Receiver)
      holds weak reference
           ↓
     Calls protocol methods
```

| Component         | Description                                                    |
| ----------------- | -------------------------------------------------------------- |
| Delegate Protocol | Defines the methods the delegate must implement                |
| Delegator (Owner) | Holds a `weak` reference to the delegate and calls its methods |
| Delegate (Client) | Implements the protocol and performs delegated tasks           |

**Example - Custom Toggle Button:**
```swift
/// Protocol
protocol ToggleButtonDelegate: AnyObject {
	func toggleButtonDidToggle(_ isOn: Bool)
}

/// Delegator
class ToggleButton: UIButton {
	weak var delegate: ToggleButtonDelegate?
	private var isOn = false

	@objc func tapped() {
		isOn.toggle()
		delegate?.toggleButtonDidToggle(isOn)
	}

	override init(frame: CGRect) {
		super.init(frame: frame)
		addTarget(self, action: #selector(tapped), for: .touchUpInside)
	}

	required init?(coder: NSCoder) {
		fatalError("init(coder:) has not been implemented")
	}
}

/// Delegate
class ViewController: UIViewController, ToggleButtonDelegate {
	override func viewDidLoad() {
		super.viewDidLoad()
		let button = ToggleButton()
		button.delegate = self
		view.addSubview(button)
	}

	func toggleButtonDidToggle(_ isOn: Bool) {
		print("Toggle state changed: \(isOn)")
	}
}
```

**Pros:**
- Decoupling - Allows class A to call class B without knowing implementation
- Custom behavior injection - Easily swap logic across multiple contexts
- Fits UIKit naturally - Foundation of table views, scroll views, etc
- Mockable/Testable - Protocols are easy to fake or stub

**Cons:**
- Retain cycles - Always declare delegate as `weak`
- Verbose - Requires boilerplate for protocol definition
- Not Swift-UI native - Combine/MVVM preferred there

**iOS Examples:**
- `UITableview` - `UITableViewDelegate`
- `UIScrollView` - `UIScrollViewDelegate`
- `UIImagePickerController` - `UIImagePickerControllerDelegate`
- `UITextField` - `UITextFieldDelegate`

**When to use:**
- UIKit views/components with repeatable callback contracts
- Parent-child VC communication
- Protocol-based architecture
- Custom UI widgets/compoents