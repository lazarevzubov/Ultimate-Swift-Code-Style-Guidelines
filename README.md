# Ultimate Swift Code Style Guidelines

These guidelines define a concrete, opinionated set of Swift style rules. Following them helps keep the codebase consistent, which improves readability, maintainability, and reduces defects.

You can adopt these rules as-is or modify them for your project. If you intentionally diverge from a rule, apply the alternative consistently across the whole codebase.

If you believe a rule is incorrect or unclear, propose a change via a pull request so the team can discuss it.

<h2 id="table-of-contents">Table of Contents</h2>

## 1. [Guidelines](#guidelines)

### 1.1. [Project](#project)

#### 1.1.1. [Files and Folders](#files-and-folders)

### 1.2. [File Structure](#file-structure)

#### 1.2.1. [Imports](#imports)

#### 1.2.2. [Members' Order](#members-order)

#### 1.2.3. [Protocol Conformances](#protocol-conformances)

### 1.3. [Explicit Documentation](#explicit-documentation)

#### 1.3.1. [Comments](#comments)

### 1.4. [Naming](#naming)

#### 1.4.1. [Types](#types)

#### 1.4.2. [Methods](#methods)

### 1.5. [API](#api)

#### 1.5.1. [Encapsulation](#encapsulation)

### 1.6. [Implementation](#implementation)

#### 1.6.1. [Methods](#implementation-methods)

#### 1.6.2. [Optionality](#optionality)

#### 1.6.3. [State and Side-Effects](#state-and-side-effects)

#### 1.6.4. [Testing](#testing)

### 1.7. [Formatting](#formatting)

#### 1.7.1. [Methods](#formatting-methods)

#### 1.7.2. [Closures](#closures)

#### 1.7.3. [Control Flow](#control-flow)

#### 1.7.4. [Literals](#literals)

#### 1.7.5. [Constants](#constants)

#### 1.7.6. [Colons](#colons)

#### 1.7.7. [Pre-processor Directives](#macros)

### 1.8. [Miscellaneous](#miscellaneous)

## 2. [Links](#links)

<h2 id="guidelines">Guidelines</h2>

<h3 id="project">Project</h3>

Do not allow compiler, linter, or analyzer warnings in the project.

Write type names, function names, documentation, and comments—everything—in American English.

<h4 id="files-and-folders">Files and Folders</h4>

Sort items alphabetically within each folder or project group: list folders first, then files.

When sorting, ignore file extensions. Name files so that related files are close to each other in the list.

Name a file after the type it contains (or the primary type if it contains multiple top-level declarations).

If a file primarily contains an extension, name it after the extended type. Optionally append `+` and the conformance name (for example, `String+ConvertibleToNumber.swift`). Omit the protocol name if the file contains multiple conformances or miscellaneous helpers (for example, `String.swift`).

[Return to Table of Contents](#table-of-contents)

<h3 id="file-structure">File Structure</h3>

Divide the file into sections using `// MARK:` comments. Separate top-level declarations (for example, two types declared at the same level) with `// MARK: -`. Group related members into sections such as `// MARK: - Methods`, and use more specific markers for subgroups (for example, `// MARK: Private methods`). Write `// MARK:` comments in grammatically correct English.

**Don't:**
```swift
//MARK:

//MARK :

// MARK: -Methods
```

Indent using spaces, because tabs may render differently across environments. Use four spaces per indentation level (two spaces is acceptable if the whole project uses it consistently).

Keep braces balanced: if you add inner spaces after `{`, add them before `}` as well. In this document, type declarations use spaced braces, and function/method bodies do not.

Separate method implementations with a single blank line. Do not insert blank lines between property declarations or between method signatures in a protocol.

End each file with a single trailing newline.

*Examples of file structure:*

**Do**:

```swift
import GatewayFoundation
import Foundation

protocol GatewayServiceDelegate {

    // MARK: - Methods

    func dataUpdated()

}

// MARK: -

protocol GatewayService {

    // MARK: - Properties

    var delegate: GatewayServiceDelegate? { get set }
    var active: Bool { get }
    var data: [GatewayDataObject] { get }

    // MARK: - Methods

    func connect()
    func disconnect()

}

// MARK: -

final class GatewayStandardService: GatewayService {

    enum GatewayType {

        case demo
        case stage

    }

    // MARK: - Properties

    // MARK: GatewayService protocol properties

    var active: Bool { server.active }
    var data: [GatewayDataObject] { server.data }
    weak var delegate: GatewayServiceDelegate?

    // MARK: Private properties

    private static let gatewayStandardAddress = URL(string: "https://standard.gateway.edu/")!
    private let serviceType: GatewayType
    private lazy var server: DataServer {
        let server = DataServer(serviceType: serviceType)
        server.delegate = self

        return server
    }

    // MARK: - Initialization

    init(serviceType: GatewayType = .stage) {
        self.serviceType = serviceType
    }

    // MARK: - Methods

    // MARK: GatewayService protocol methods

    func connect() {
        try? server.establishConnection()
    }

    func disconnect() {
        server.breakConnection()
    }

}

// MARK: - DataServerDelegate

extension GatewayStandardService: DataServerDelegate {

    // MARK: - Methods

    // MARK: GatewayStandardService protocol methods

    func connectionEstablished() {
        notifyDelegate()
    }

    func dataUpdated() {
        notifyDelegate()
    }

    // MARK: Private methods

    private func notifyDelegate() {
        delegate?.dataUpdated()
    }

}
```

**Don't**:

```swift
import Foundation // FIXME: Wrong imports order
import GatewayFoundation
import UIKit // FIXME: Unused import
import Darwin // FIXME: Excessive import – Foundation includes Darwin.

// FIXME: Extra empty line
protocol GatewayServiceDelegate { // FIXME: Absent section dividers

  func dataUpdated() // FIXME: Broken indentation

}
protocol GatewayService { // FIXME: Absent file structural divider and an empty line

    // MARK: - Properties

    var active: Bool { get }
    var data: [GatewayDataObject] { get }
    var delegate: GatewayServiceDelegate? { get set } // FIXME: Less strict accessible properties should go first.

    // MARK: - Methods

    func connect()
    func disconnect()
} // FIXME: Absent empty line before type declaration closing brace

// MARK: -

final class GatewayStandardService: GatewayService, DataServerDelegate { // FIXME: Additional conformances should be declared in extensions.

    // MARK: - Properties

    // MARK: GatewayService protocol properties

    private static let gatewayStandardAddress: URL = URL(string: "https://standard.gateway.edu/")! // FIXME: Private and "public" members are mixed.
    private let serviceType: GatewayType
    weak var delegate: GatewayServiceDelegate?
    var data: [GatewayDataObject] {
        return server.data // FIXME: The "return" keyword in a single-statement body.
    }
    var active: Bool { // FIXME: Properties of the same access level should be sorted alphabetically.
        server.active // FIXME: Single-statement scopes are preferred to be placed on a single line with both braces.
    }
    private lazy var server: DataServer {
        let server = DataServer(serviceType: serviceType)
        server.delegate = self

        return server
    }

    // MARK: - Initialization

    init(serviceType: GatewayType = .stage) {
        self.serviceType = serviceType
    }

    // MARK: - Methods

    // MARK: GatewayService protocol methods

    func connect() {
        try? server.establishConnection()
    }

    func disconnect() {

        server.breakConnection()

    } // FIXME: Empty lines in the beginning and the end of the method.

    // MARK: GatewayStandardService protocol methods

    func connectionEstablished() {
        notifyDelegate()
    }

    func dataUpdated() {
        notifyDelegate()
    }

    // MARK: Private methods

    private func notifyDelegate() {
        delegate?.dataUpdated()
    }

    enum GatewayType { // FIXME: Nested types accessible from the outside should go first.
        case demo
        case stage
    } // FIXME: No spaces after the opening and before the closing brace.

}
```

Here's another short example of good file structure, a SwiftUI View.

**Do**:

```swift
import SwiftUI

struct ContentView: View {

    // MARK: - Properties

    // MARK: View protocol properties

    var body: some View {
        VStack {
            Text("Hello, \(name)!")
            Text("And have a nice day.")
        }
            .frame(maxWidth: 200.0)
    }

    // MARK: Private properties

    @State
    private var name = "Steve"

}
```

Do not enforce a hard limit on file length. Keep files small by following modern design principles (for example, SOLID, DRY, KISS, and YAGNI). As a rule of thumb, treat ~200 lines (including formatting and comments) as a review prompt—not a strict limit. Well-structured files longer than 200 lines can still be acceptable; use your judgment.

<h4 id="imports">Imports</h4>

Place all imports at the top of the file. Put one import per line, sort them lexicographically, and do not insert blank lines between them.

Import the narrowest module(s) you actually use. Avoid redundant imports (for example, do not import `Foundation` if you already import `UIKit`).

<h4 id="members-order">Members' Order</h4>

Order properties within a subgroup (and enum cases) first by access level, then by `static` vs instance membership, then lexicographically.

Logical ordering is acceptable but not encouraged. Use it only when all neighboring declarations form a clear logical sequence. Do not mix logical and lexicographical ordering within the same subgroup.

Root declarations start from the beginning of the line. Each level of nesting adds a step of indentation.

Order public and internal methods logically (for example, by lifecycle and call order).

**Do**:

```swift
func loadView() {
    // ...
}

func viewDidLoad() {
    // ...
}

func viewWillLayoutSubviews() {
    // ...
}

func viewDidLayoutSubviews() {
    // ...
}

func viewWillAppear() {
    // ...
}

func viewDidAppear() {
    // ...
}

func viewWillDisappear() {
    // ...
}

func viewDidDisappear() {
    // ...
}

```

Order private methods by first use (methods called earlier come first).

Avoid mixing public and private APIs. However, placing methods in order of their usage and closer to the first calling (i.e., "as an engineer would read" the source file in order to understand the flow) is acceptable.

<h4 id="protocol-conformances">Protocol Conformances</h4>

Add protocol conformances in separate extensions unless it's the main role of the type.

**Do**:

```swift
protocol Presenter {
    // ...
}

struct DefaultPresenter: Presenter {
    // ...
}
```

```swift
protocol Presentable {
    // ...
}

final class ViewController: UIViewController {
    // ...
}

extension ViewController: Presentable {
    // ...
}
```

**Don't**:

```swift
protocol Presenter {
    // ...
}

struct DefaultPresenter {
    // ...
}

extension DefaultPresenter: Presenter {
    // ...
}
```

Protocol extensions go after the main declaration and are separated by a ```// MARK: -``` comment with the protocol name. It results in a nice "table of contents" in Xcode's file navigator:

**Do**:

```swift
struct SomeType {

    // ...

}

// MARK: - DifferentProtocol

extension SomeType: DifferentProtocol {

    // MARK: - Methods

    // MARK: DifferentProtocol protocol methods

    // ...

}
```

[Return to Table of Contents](#table-of-contents)

<h3 id="explicit-documentation">Explicit Documentation</h3>

Each non-private API should have a [HeaderDoc](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/HeaderDoc/intro/intro.html) comment, even if the API is well self-documented. The main assignment of having exhaustive documentation of public APIs is to be visible from different code base places through IDE's help features, such as quick help popovers.

Private APIs should be self-documented and have no explicit documentation in order not to distract from reading implementation details.

Indicating a method's time complexity in documentation generally is a good idea, unless irrelevant.

The basic structure of multi-line descriptions:

**Do:**

```swift
/// Summary.
///
/// Detailed description.
///
/// - Parameters:
///     - parameter parameter1: Parameter description.
///     - parameter parameter2: Parameter description.
/// - Returns: Return value description.
/// - Throws: Exception description.
```

Don't use multi-line documentation put between `/**` and `*/`, because the former way is generated by Xcode by default and it's shorter.

**Don't:**

```swift
/**
Summary.

Detailed description.

- Parameter parameter1: Parameter description.
- Parameter parameter2: Parameter description.
- returns: Return value description.
- Throws: Exception description.
*/
```

Everything but summary is optional. The order is as follows:

- Summary
- Discussion
- `Parameter`/`Parameters`
- `Returns`
- `Throws`

Though, if a method has any parameters, they are necessary to be documented. Document a single parameter as `- Parameter: Description.` Multiple parameters – by the help of `- Parameters:` syntax. If the method has functions as parameters, their arguments must have labels and be documented, as well.

Document a method's return value type if it is not `Void`.

If a method throws an exception, document the exception with `Throws`.

Divide each two HeaderDoc parts by a single blank line. However, if there's no description, blank lines are omitted:

**Do:**

```swift
/// Summary.
/// - Parameter parameter: Parameter description.
/// - Returns: Return value description.
/// - Throws: Exception description.
```

**Don't:**

```swift
// Summary.
//
// - Parameter parameter: Parameter description.
//
// - Returns: Return value description.
//
// - Throws: Exception description.
```

Use the [Apple Markup](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/) syntax for links, code snippets, etc..

Links to related methods are encouraged and implemented by means of tags. That is, the referenced method is marked with `- Tag: SomeTag` and the referencing method uses it as this: `[click here](x-source-tag://SomeTag)`.

Don't add blank lines between a documentation comment and the documented member.

<h4 id="comments">Comments</h4>

Prefer self-documenting code over comments.

If you change the code a comment refers to, update the comment or delete it.

Start single-line comments with `//`. Use `///` only for documentation comments (HeaderDoc/DocC).

Start multi-line comments with `/*` and end them with `*/`. Put the delimiters on their own lines, and do not add blank lines immediately after `/*` or immediately before `*/`.

Place comments on the line above the commented code. Separate a comment from the preceding code with a blank line (except when the comment is the first line in a scope). You may place a short single-line comment at the end of a code line if it does not make the line too long.

[Return to Table of Contents](#table-of-contents)

<h3 id="naming">Naming</h3>

All declarations should be self-explanatory and consist of American English words and form correct human-readable phrases.

Everything but types is `lowerCamelCase`. The types are `UpperCamelCase` (a.k.a. `PascalCase`.)

Abbreviations don't have different cases within them and are cased up or down according to the rule above (e.g., `let asciiTable: [String : String]`, `class SMTPServer`). (A common mistake is to use `Id` instead of `ID` – a convention, common in other programming languages.)

Do not use underscores, even at the beginning of property names. Do not use Hungarian notation for constants (for example, a leading "k").

In most cases, variables have noun phrase names: `let number: Int`. The most significant exception is booleans which are expected to have assertive names (e.g., `let necessary: Bool`). Boolean names are preferred without a verb at the beginning, unless it introduces ambiguity:

 **Do:**

```swift
let necessary: Bool
```

**Don't:**

```swift
let isNecessary: Bool
```

<h4 id="types">Types</h4>

Name classes and structures using noun phrases, as well as protocols describing what an object is. Name protocols which add abilities descriptively (e.g., `Sortable`). Protocols should not have the word `Protocol` at the end of the name. Instead, conforming types should have a specifying word in the name, because protocols are general, but classes and structures are specific.

**Do**:

```swift
protocol Engine { }
struct DefaultEngine: Engine { }
```

**Don't**:

```swift
protocol EngineProtocol { }
struct Engine: EngineProtocol { }
```

Types implementing design patterns are usually named with the pattern name at the end (e.g., `ViewBuilder`, `DisplayingStrategy`).

Don't use prefixes (e.g., just `PriceCalculator` instead of `XYZPriceCalculator`), since there's no necessity for that in Swift (other than, maybe, to maintain consistency with Objective-C libraries and components).

<h4 id="methods">Methods</h4>

Name methods without side-effects in the form of what: `let size = view.originalSize()`.

Name methods with side effects in the form of an imperative verb: `list.sort()`.

Similarly, name non-mutating methods that return new values using past participle: `let newList = oldList.sorted()`. Or present participle: `let newList = oldList.sortingLexicographically()`.

Parameter names follow rules for variable names. Make use of Swift syntax possibilities to make the full declaration a human-readable phrase. For example, `func insert(_ element: Element, at index: Int)` is nicely read as `insert(someElement, at: someIndex)` at the call site.

Start factory method names with the word "make". Both factory methods and initializers have their parameters as a list of items, they are not required to form phrases.

**Do**:

```swift
init(name: String, id: Int)
func makeView(position: CGPoint, size: CGSize) -> UIView
```

**Don't**:

```swift
init(withName name: String, andID id: Int)
```

It's very common to force engineers to put an object of delegation as the first argument of delegation methods. This is not strictly necessary and is used only in cases when it makes sense.

**Do**:

```swift
func buttonTapped(_ button: UIButton)
```

**Don't** (unless the object of delegation is really needed at the call site:)

```swift
func screen(_ screen: UIViewController, hasButtonTapped button: UIButton)
```

Name UIKit's `UIControl` actions with the control's name in the beginning and the "action" word in the end:

**Do**:

```swift
@objc
private func nextButtonAction(_ sender: UIButton) { // ...
```

(The sender argument is optional since it's often unused.)

**Don't**:

```swift
@objc
private func onNextButtonTapped(_ sender: UIButton) { // ...
```

The latter naming convention is confusing because makes you think of delegation. However, actions may be thought of as a kind of delegation, and the naming style might make sense for some code bases.

[Return to Table of Contents](#table-of-contents)

<h3 id="api">API</h3>

Prefer `struct` for value types. Prefer `class` for reference types (for example, objects with identity, state, or a lifecycle). Other considerations (semantics, memory management, and so on) are out of scope for this document.

Keep properties constant-time. If a computed property is not constant-time, convert it to a method.

Avoid optional booleans and collections, because they bring degraded states: what's the difference between an empty array and a `nil` array within the same context?

Avoid free functions. Double-check whether the function should belong to a type.

Do not return the “object of the action” from a command-style method (for example, do not return the presented view controller from a `present…` method).

**Do**:

```swift
func presentErrorAlert()
```

**Don't**:

```swift
func presentErrorAlert() -> UIAlertController
```

However, if the return value might be useful in some specific situations, it doesn't force one to use it (the `@discardableResult` annotation serves this purpose). Though, such return values are not encouraged. Follow the principle of [the command-query separation](https://en.wikipedia.org/wiki/Command–query_separation).

Avoid abbreviations and acronyms (except for the common ones). (Possible exceptions are local variables within implementation blocks of code.)

**Do**: `let currentValue = 1`

**Don't**: `let curValue = 1` (Though, it's acceptable in a very limited scope, like a method implementation).

<h4 id="encapsulation">Encapsulation</h4>

Add `final` to classes by default. Remove it only when you explicitly need inheritance. (Prefer composition over inheritance.)

`fileprivate` access modifier is usually a code smell. Reconsider an alternative design approach (e.g., nested types).

Don't specify the default access modifier, namely `internal`, explicitly (as well as any other default, in general).

`class` type members are rarely useful because of discouraged use of inheritance, especially for static members. Be aware of the use-cases of `class` and `static` modifiers (confusing them is a common mistake.)

Generally, honor encapsulation in any way. E.g., always declare `@IBOutlet` properties and `@IBAction` methods `private`. Hide implementation details behind meaningful API.

[Return to Table of Contents](#table-of-contents)

<h3 id="implementation">Implementation</h3>

Use type inference and the current namespace. The compiler notifies them when a type should be stated explicitly. However, there might be occasions when specifying a type explicitly makes compilation faster.

Similar rules are applied to nested types and enum cases. The shorthand notation starting with a dot is preferred whenever is possible.

Use lazy initialization for properties that are not immediately needed. Also prefer complex lazily initialized properties for subviews of UIKit's views and view controllers, especially over configuring them within view controller's lifecycle methods (a common mistake is to write big and awkward `viewDidLoad()` implementations consisting of the full view controller's initial configuration.)

Check the “golden path” (the most likely case) first. Avoid inverted checks (for example, `!incorrect`) because they reduce readability. If these rules conflict, refactor the condition for clarity.

**Initialization**

If the initial or constant value of a property doesn't depend on the initializer's parameters, prefer a default value over setting it within the initialization code.

`.init()` is not used for initialization:

**Do**:

```swift
let color = UIColor(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
```

**Don't**:

```swift
let color: UIColor = .init(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
```

<h4 id="implementation-methods">Methods</h4>

Any scope contains only elements of the same level of abstraction (i.e., either variables and related direct calculations, or method calls of the same layer of abstraction).

**Do**:
```swift
func viewDidLoad() {
    super.viewDidLoad()
    presenter.viewLoaded()
}
```

**Don't**:
```swift
func viewDidLoad() {
    super.viewDidLoad()

    let button = UIButton()
    button.addTarget(self, action: #selector(buttonAction), event: .touchUpInside)
    layoutButton()

    view.backgroundColor = .black
}
```

Methods, as well as any code part, follow the Single Responsibility Principle. Beside maintainability, this doesn't let methods grow long and difficult to read. However, as for length, there might be exceptions. For example, complex drawing methods that can't be broken up into several small ones (or breaking up would make the code less readable.)

<h4 id="optionality">Optionality</h4>

Avoid force-unwraps. Basically, if an object is optional, that's because it really might be `nil` in certain conditions. If you are 100% sure (which is usually impossible) that the optional value can't be `nil`, it's better to state your point of view explicitly. For instance, by adding some kind of assertion inside a `guard` statement.

A possibly justified reason to have a force-unwrapped variable is late initialization. However, the latter may also be error-prone and should be avoided.

Another acceptable reason to have a force-unwrap is an optional API which returns `nil` only if there's a programming error (e.g., creating an URL from `String` in Foundation: `let url = URL(string: "https://www.apple.com")!`). However, you should not rely on third-party APIs' implementation details since they might change at any time and without notice.

In testing code, force-unwraps are less discouraged because if they fail, the test fails too, and this is desired. However modern testing helpers like Testing's `#require` macro should be preferred.

<h4 id="state-and-side-effects">State and Side Effects</h4>

Prefer clean functions over state-changing ones. Prefer unidirectional flow over side effects. It makes code less error-prone, improves readability and testability.

**Don't**:
```swift
private var button: UIButton!

// ...

func viewDidLoad() {
    super.viewDidLoad()
    button = UIButton()
    configureButton()
}

// ...

private func configureButton() {
    // ...
}
```

In the example above it's hard to track the actions' sequence.

**Do**:

```swift
private var button: UIButton! {
    didSet { configure(button: button) }
}

// ...

func viewDidLoad() {
    super.viewDidLoad()
    button = UIButton()
}

// ...

private func configure(button: UIButton) {
    // ...
}
```

In the latter example, `button` is initialized once, within the `viewDidLoad()` method. Once it's set, `configure(button:)` is called – a pure function that doesn't rely on any kind of state and has no side effects. That is, it always results in the same way for the same arguments.

<h4 id="testing">Testing</h4>

Test interfaces, not implementation: call public APIs and assert expected output for a given input.

Do not break encapsulation for testing. If anything is wanted to be overridden or substantiated in testing code, protocols and their mock or stub implementations solve the problem.

[Return to Table of Contents](#table-of-contents)

<h3 id="formatting">Formatting</h3>

Avoid multiple variable declarations on a single line using a single `var`/`let` keyword (for example, `var x: Int, y: Int`). Do not sacrifice readability for brevity.

Don't use custom horizontal alignment because it welcomes further maintenance problems:

**Don't**:

```swift
let top:    CGPoint
let middle: CGPoint
let bottom: CGPoint
```

**Type aliases**

Only use `typealias` declarations for the sake of brevity when it doesn't prevent clarity.

**Do**:

```swift
typealias Task = (_ token: String) -> (_ result: Result<Data, Error>)
func enqueue(task: Task)
```

**Don't**:

```swift
typealias T = (Int, Int) -> (String)
func process(task: T) -> String
```

<h4 id="formatting-methods">Methods</h4>

Place a method declaration on a single line if it can fit most display screen widths without a carry-over. Otherwise, place each parameter on its own line, matching the beginning of the previous one. Carry return type on to the last parameter's line.

**Do**:

```swift
func fetchResults(
    from endpoint: URL = .remoteServerPath,
    transferringTo device: Device = .current,
    compressed: Bool = true,
    completionHandler: ((_ success: Bool) -> ())? = nil
) -> [Data]
```

**Don't**:

```swift
func fetchResults(from endpoint: URL,
                  transferringTo device: Device,
                  compressed: Bool,
                  completionHandler: (() -> Void)?) -> [Data]

func fetchResults(from endpoint: URL, transferringTo device: Device,
                  compressed: Bool, completionHandler: (() -> Void)?) -> [Data]
```

In the former example, the formatting will be broken if the function is renamed. In the latter example, parameters, which go second on the lines are hard to notice.

Bottom line: here's the prioritized order, in which the arguments are formatted, depending on the available space:

**Do**:

```swift
func doSomething1(argument1: Int, argument2: Int)

func doSomething3(
    argument1: Int, argument2: Int, argument3: Int, argument4: Int
)

func doSomething4(
    argument1: Int,
    argument2: Int,
    argument3: Int,
    argument4: Int,
    argument5: Int
)
```

**Custom operators**

Add space to operator declarations between the implemented operator's name and the left parens.

**Do**:

```swift
static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
```

**Line length**

Use Java Code Conventions as a reference for maximum line length, or use common sense (typically 80–120 characters per line).

**Annotations**

Put attributes starting with a `@` symbol on separate lines before the corresponding declarations.

**Do**:

```swift
@testable
import ServiceFoundation
```

```swift
@objc
private func acceptanceButtonAction(_ sender: UIButton) {
```

**Line breaks**

Break long expressions into several parts on different lines so that the symbol that connects two parts of expression starts the new line. Indent each new line with one additional indentation level. Having a special character starting a new line avoids creating the illusion that a new line is a new statement.

**Do**:
```swift
let a = (a + b)
    + (a + c)
```

**Don't**:
```swift
let a = (a + b) +
    (a + c)
```

**Return**

Separate code within the scope (e.g., method's implementation) into logical blocks by empty lines. Don't use any separation, if all blocks are one-line. `return` statement is usually separated, the only exception is if there's only two lines overall.

**Do**:

```swift
func makeOrderViewController(orderNumber: Int) -> UIViewController {
    guard let order = Order(orderNumber: Int) else {
        fatalError("Non-existent order.")
    }

    let model = OrderModel(orderNumber: orderNumber)
    let vc = OrderViewController(model: model)

    return vc
}
```

```swift
func makeOrderViewController(orderNumber: Int) -> UIViewController {
    let model = OrderModel(orderNumber: orderNumber)
    let vc = OrderViewController(model: model)

    return vc
}
```

```swift
func makeOrderViewController(model: OrderModel) -> UIViewController {
    let vc = OrderViewController(model: model)
    return vc
}
```

**Don't**:

```swift
func makeOrderViewController(orderNumber: Int) -> UIViewController {
    let model = OrderModel(orderNumber: orderNumber)
    let vc = OrderViewController(model: model)
    return vc // FIXME: The return statement is not separated.
}
```

```swift
func makeOrderViewController(model: OrderModel) -> UIViewController {
    let vc = OrderViewController(model: model)

    return vc // FIXME: Senseless blank line between the two lines.
}
```

Omit the `return` keyword, if the implementation consists only of a return statement:

**Do**:

```swift
func getViewController() -> UIViewController {
    vc
}
```

<h4 id="closures">Closures</h4>

Prefer trailing closure syntax anywhere possible. Therefore, don't overload methods with closures as arguments differing only by the name of the trailing closures – that leads to ambiguity on the call site.

Don't use empty parentheses on the call site for the methods with the only closure as an argument.

**Do**:

```swift
requestMoreMoney { spend($0) }
```

**Don't**:

```swift
requestMoreMoney() { spend($0) }
```

```swift
requestMoreMoney(completionHandler: { spend($0) })
```

Prefer the functional style whenever possible: omit explicit argument names (use placeholders like `$0`), place one-line closure content on the same line with braces (with respect to the line width limits, though).

**Don't**:

```swift
requestMoreMoney { money in
    spend(money)
}
```

Break chained functional-style calls into several lines before a dot, if the call doesn't fit a single line. Don't leave the first call of the chain on the first line. Indent each new line of this call with exactly one extra level of indentation.

**Do**:

```swift
array
    .filter { $0 > 0 }
    .sort { $0 > $1 }
    .map { Int($0) }
```

**Don't**:

```swift
array.filter { $0 > 0 }
    .sort { $0 > $1 }.map { Int($0) }
```

The latter snippet gives a false impression of only two elements in the call chain.

Use one indentation level also after parentheses.

**Do**:
```swift
Button(action: proceed) {
    Text("Proceed")
}
    .frame(maxWidth: 200.0)
```

**Don't**:
```swift
Button(action: proceed) {
    Text("Proceed")
}
.frame(maxWidth: 200.0)
```

<h4 id="control-flow">Control Flow</h4>

Open braces on the same line with declaration and close them on a separate line. Don't leave empty lines after the opening brace and before the closing one. (The only exception is functional-style calls or, say, property observers which syntactically might be considered as functional statements. See the Methods and Closures sections.)

**Do**:

```swift
if name.isEmpty {
    router.showSettings()
} else {
    router.showProfile()
}
```

**Don't**:

```swift
if name.isEmpty
{
    router.showSettings()
}
else
{
    router.showProfile()
}
```

```swift
if name.isEmpty {
    router.showSettings()
}
else {
    router.showProfile()
}
```

```swift
if name.isEmpty {

    router.showSettings()

} else {

    router.showProfile()

}
```

Place multiple conditions of a single `if`-statement on separate lines. Prefer using commas instead of logic "and" (`&&`).

**Do**:

```swift
if password.isCorrect,
    user.exists {
    // ...
```

**Don't**:

```swift
if password.isCorrect, user.exists {
    // ...
```

```swift
if password.isCorrect
    && user.exists {
    // ...
```

If multiple paths exist, put the most likely one first.

In any `guard`-statement, the `else` (and its left brace) goes on the same line after the last condition.

**Do**:

```swift
guard !array.isEmpty else {
    // ...
```

**Don't**:

```swift
guard !array.isEmpty
    else {
    // ...
```

Distinguish multiple and complex logical conditions with braces. Place distinguished conditions on their own lines, with corresponding indentation:

`let goodToGo = ((firstNumber > 0)
        || (firstNumber < -10)) // Two levels of indentation, because it's a sub-condition of the first part of the main condition expression.
    && isFirstTry`. // One level of indentation, because it's the second part of the main condition expression.

Avoid unnecessary parentheses within `if`-statement conditions:

**Do**:

```swift
if happens {
    // ...
```

**Don't**:

```swift
if (number != 0) {
    // ...
```

If a control-flow keyword is followed by a left parenthesis, insert exactly one space between them.

Don't use the ternary operator `?`-`:` where a single `if`-check is sufficient, because although it can save lines, it makes the intention unclear and spawns extra entities (empty tuples or functions).

**Do**:

```swift
if error == nil {
    completion()
}
```

**Don't**:

```swift
error == nil
  ? completion()
  : ()
```

Place ternary operator expressions on three lines, with line breaks before the first and the second symbols of the operator and one extra indentation level before the operator signs.

Prefer `for … where` clauses over `if` statements nested inside loops.

Prefer clearly named, easy-to-understand local constants over inlined conditions:

**Do**:

```swift
let withinSolarSystem = ((2 * 2) == 4)
if withinSolarSystem {
    // ...
```

**Don't**:

```swift
if (2 * 2) == 4 {
    // ...
```

**Switch Statements**

Use one level of indentation inside a `switch`'s parentheses and for `case` implementations.

Start all statements inside the cases of a `switch` statement on separate lines.

**Do**:

```swift
switch direction {
    case .left:
        turnLeft()
    case .right:
        turnRight()
    case .straight:
        break
}
```

**Don't**:

```swift
switch direction {
    case .left: turnLeft()
    case .right: turnRight()
    case .straight: break
}
```

Don't use `default` cases (unless it's an imported, non `NS_CLOSED_ENUM`, Objective-C enumeration) because they may lead to erroneous states if new cases are added.

<h4 id="literals">Literals</h4>

**Arrays**

Don't add space after the left square bracket and before the right one of array literals. List the included items one below another, aligned at the same level of indentation. Put the first element on the line right next after the declaration. Put the closing bracket on the next line after the last item. However, if items are short and their sequence can be read easily (e.g., integer literals) it's acceptable to have them all on the one line.

**Do**:

```swift
var numbers = [1, 2, 3]

let airVehicles = [
    helicopter,
    airLiner,
    carrierRocket,
    wings
]
```

**Don't**:
```swift
var numbers = [1,
               2,
               3]

let airVehicles = [helicopter, airLiner, carrierRocket, wings]
```

Use the option with braces on separate lines when elements don't fit the line width:

**Do**:

```swift
let airVehicles = [
    assumeThisNameDoesNotFitTheLineWidth,
    airLiner
]
```

Don't use the trailing comma after the last element.

**Strings**

Prefer multi-line strings over concatenation.

**Do**:

```swift
let fullName = """
               Nikita
               Lazarev-Zubov
               """
```

**Don't**:

```swift
let fullName = "Nikita\n"
    + "Lazarev-Zubov"
```

An exception is `NSLocalizedString`, because the genstrings tool cannot handle Swift's multi-line strings.

**Numbers**

Separate groups of digits in long numbers with underscores: `1_000_000_000`

Always use an explicit decimal fraction with floating-point numbers, even when it's zero. Breaking this rule leads to ambiguity when decimal number variables are assigned with new values.

**Don't**:

```swift
let coordinate: CGFloat = 2.1
// ...
coordinate = 2 // Without looking at the declaration it's impossible to know that it's a floating point number.
```

<h4 id="constants">Constants</h4>

Avoid global constants.

Group the constants within a type declaration logically into private case-less `enum`s as their static properties. Using case-less `enum`s instead of structures or classes prevents unwanted initialization of the containing entity, without adding an explicit private empty initializer.

<h4 id="colons">Colons</h4>

Colons always have no space on the left and one space on the right. Exceptions are ternary operators (`?`-`:`) and dictionaries.

**Do**: `let scoreTable: [String : Int] = [:]`

**Don't** (three of them in a row – spot them all): `let scoreTable : [String: Int] = [ : ]`

Surround the ampersand in composition statements (`Codable & Encodable`) by spaces too.

**Operators**

All operators (including `=`, but excluding the already discussed `:`) have exactly one standard space before and after.

**Do**: `let a = b + c`

**Don't**: `if (b+c) == 1 {`

<h4 id="macros">Preprocessor Directives</h4>

Don't indent macros, format the surrounded code as if the macro doesn't exist.

**Do**:

```swift
func handleLogin() {
#if DEBUG
    printDebugInfo()
#endif
    openHomeScreen()
}
```

**Don't**:

```swift
func handleLogin() {
    #if DEBUG
        printDebugInfo()
    #endif
    openHomeScreen()
}
```

[Return to Table of Contents](#table-of-contents)

<h3 id="miscellaneous">Miscellaneous</h3>

Don't use semicolons. The only case in Swift when they are necessary is multiple statements on a single line, which are strongly discouraged.

Use only the Latin alphabet and numbers in the source code (no Cyrillic alphabet, hieroglyphs, emojis, etc.). Although the Swift programming language supports any UTF symbols, they are hard to type using some of the various localized keyboards and encourage error-prone copy/paste.

Remove unused code (including imports and resources), as well as empty or inherited method implementations.

Empty blocks usually indicate a design problem. If you cannot avoid an empty block, add a comment explaining why it is empty:

**Do**:

```swift
func buttonAction() {
    // The button is inactive in this state and does not need an action.
}
```

**Void**

`()`, `Void` and `(Void)` are syntactically interchangeable. However, the first one means an empty tuple that can be used, for example, as empty list of arguments of a closure. The second one is used as an empty return from a closure. The latter is just a pointless usage of a possibility to put any statement between parentheses and is not used.

**Do**: `let completionHandler: () -> Void`

**Don't**: `let completionHandler: () -> ()`

**Access modifiers**

Access modifiers of types, methods etc. go first.

**Do**: `public static func goPublic()`

**Don't**: `static public func goPublic()`

[Return to Table of Contents](#table-of-contents)

<h2 id="links">Links</h2>

[**Finding the Ultimate Swift Code-Style Guidelines**](https://betterprogramming.pub/finding-the-ultimate-swift-code-style-guidelines-59025a7c163c)

A brief introduction to these guidelines.

[**Swift.org API Design Guidelines**](https://swift.org/documentation/api-design-guidelines/)

Non-negotiable official guidelines, focused mainly on API and naming.

[**Java Code Conventions**](https://oracle.com/technetwork/java/codeconvtoc-136057.html)

An inspiring example of an exhaustive code style for a programming language. Though, it has many atavisms and old-fashioned rules. It's really surprising to come across such an ugly formatting every now and then:

**Don't**:

```java
void layoutScreen() {
    layoutHeader()

    layoutFooter()
}
```

[**Steve McConnell – Code Complete: A Practical Handbook of Software Construction**](https://en.wikipedia.org/wiki/Code_Complete)

An impressive work, guiding software development in general, paying a lot of reader's attention on the code style, exhaustive explanations included.
