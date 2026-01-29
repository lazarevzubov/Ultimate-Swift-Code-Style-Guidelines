# Ultimate Swift Code Style Guidelines

These guidelines define a concrete, opinionated set of Swift style rules. Following them helps keep the codebase consistent, which improves readability, maintainability, and reduces defects.

You can adopt these rules as-is or modify them for your project. If you intentionally diverge from a rule, apply the alternative consistently across the whole codebase.

If you believe a rule is incorrect or unclear, propose a change via a pull request so the team can discuss it.

<h2 id="table-of-contents">Table of Contents</h2>

## 1. [Guidelines](#guidelines)

### 1.1. [Project](#project)

#### 1.1.1. [Files and Directories](#files-and-directories)

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

<h4 id="files-and-directories">Files and Directories</h4>

Sort items alphabetically within each folder or project group: list directories first, then files.

When sorting, ignore file extensions. Name files so that related files are close to each other in the list.

Name a file after the type it contains (or the primary type if it contains multiple top-level declarations).

If a file primarily contains an extension, name it after the extended type. Optionally append `+` and the conformance name (for example, `String+ConvertibleToNumber.swift`). Omit the protocol name if the file contains multiple conformances or miscellaneous helpers (for example, `String.swift`).

[Return to Table of Contents](#table-of-contents)

<h3 id="file-structure">File Structure</h3>

File is divided into sections by `// MARK:` comments. Root declarations (e.g. two classes on the same level) are separated by `// MARK: -`. Declarations within the same group of a type are gathered together in sections, like `// MARK: - Methods`. Multiple declarations within a group – like `// MARK: Private methods`. This type of comments (as well as any other) is kept close to grammatically correct language.

**Don't:**
```swift
//MARK:

//MARK :

// MARK: -Methods
```

Indentation shall be made with spaces, because tab symbols can be represented differently in different environments. One level of indentation is four spaces. Two spaces is an acceptable option.

Keep braces balanced: if you add inner spaces after `{`, add them before `}` as well. In this document, type declarations use spaced braces, and function/method bodies do not.

Separate method implementations with a single blank line. Do not insert blank lines between property declarations or between method signatures in a protocol.

Files shall end with a single blank line.

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
        return server.data // FIXME: Single statements shall not use the "return" keyword.
    }
    var active: Bool { // FIXME: Properties of the same access level should be sorted alphabetically.
        server.active // FIXME: Single statement scopes are preferred to be placed on a single line with both braces.
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

File length doesn't have a specific line number limitation, since basically, code shall respect contemporary software development principles (mainly, S.O.L.I.D., but also D.R.Y., K.I.S.S., Y.A.G.N.I., and so on.) Following the principles won't let an engineer create an unacceptably long file. However, if an engineer isn't experienced and doesn't have an experienced reviewer, the number of 200 lines (including all formatting and comments) could be used for guidance. Though, this doesn't mean that well organized files of the length of 201 or even 300 lines are necessarily evil – an engineer shall be guided by their common sense. 

<h4 id="imports">Imports</h4>

Place all imports at the top of the file. List one import per line, sort them lexicographically, and do not insert blank lines between them.

Import the narrowest module(s) you actually use. Avoid redundant imports (for example, do not import `Foundation` if you already import `UIKit`).

<h4 id="members-order">Members' Order</h4>

Properties within a subgroup, as well as enum cases, shall be ordered, firstly, by its access modifier, then by the class/instance membership type, then lexicographically.

Logical sorting is acceptable, but not encouraged and may be used only if all neighboring declarations can be sorted logically. Partially logical and partially lexicographical ordering is not used.

Root declarations start from the beginning of the line. Each level of nesting adds a step of indentation.

Public and internal methods shall be sorted in a logical order. It's always possible to decide on one for methods. For example, by remembering the type's lifecycle and methods' calling order.

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

Private methods are sorted by their first mention (mentioned earlier go first).

Mixing public and private APIs shall be strictly avoided. However, placing methods in order of their usage and closer to the first calling (i.e., "as an engineer would read" the source file in order to understand the flow) is acceptable.

<h4 id="protocol-conformances">Protocol Conformances</h4>

Protocol conformances are added within separate extensions unless it's the main role of the type.

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

Multi-line documentation put between `/**` and `*/` is not used, because the former way is generated by Xcode by default and it's shorter. 

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

Everything but summary is optional. The order shall be:
- Summary 
- Discussion
- `Parameter`/`Parameters`
- `Returns`
- `Throws` 

Though, if a method has any parameters, they are necessary to be documented. A single parameter must be documented as `- Parameter: Description.` Multiple parameters shall be documented by the help of `- Parameters:` syntax. If the method has functions as parameters, their arguments must have labels and be documented, as well.

If a method's return value type is not `Void`, it should be documented.

If a method throws an exception, the exception is documented with `Throws`.

Two HeaderDoc parts are divided by a single blank line. However, if there's no description, blank lines are omitted:

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

Links, code snippets, and such shall make use of the [Apple Markup](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/) syntax.

Links to related methods are encouraged and implemented by means of tags. That is, the referenced method is marked with `- Tag: SomeTag` and the referencing method uses it as this: `[click here](x-source-tag://SomeTag)`.

There's no blank lines between a documentation comment and the documented member.

<h4 id="comments">Comments</h4>

Prefer self-documenting code over comments.

If you change commented code, update the comment or delete it.

Start single-line comments with `//`. Use `///` only for documentation comments (HeaderDoc/DocC).

Start multi-line comments with `/*` and end them with `*/`. Put the delimiters on their own lines, and do not add blank lines immediately after `/*` or immediately before `*/`.

Place comments on the line above the commented code. Separate a comment from the preceding code with a blank line (except when the comment is the first line in a scope). You may place a short single-line comment at the end of a code line if it does not make the line too long.

[Return to Table of Contents](#table-of-contents)

<h3 id="naming">Naming</h3>

All declarations should be self-explanatory and consist of American English words and form correct human-readable phrases.

Everything but types is `lowerCamelCase`. The types are `UpperCamelCase` (a.k.a. `PascalCase`.)

Abbreviations don't have different cases within them and are cased up or down according to the rule above (e.g., `let asciiTable: [String : String]`, `class SMTPServer`). (A common mistake is to use `Id` instead of `ID` – a convention, common in Java.)

No underscores, even at the beginning of property names, shall be used. No Hungarian notation for constants (e.g., leading "k") either.

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

Classes and structures are named using noun phrases, as well as protocols describing what an object is. Protocols which add abilities are named descriptively (e.g., `Sortable`). Protocols should not have the word `Protocol` at the end of the name. Instead, conforming types should have a specifying word in the name, because protocols are general, but classes and structures are specific.

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

No prefixes shall be used (e.g., just `PriceCalculator` instead of `XYZPriceCalculator`), since there's no necessity for that in Swift (other than, maybe, to maintain consistency with Objective-C libraries and components).

<h4 id="methods">Methods</h4>

Methods without side-effects have names in the form of what: `let size = view.originalSize()`.

Methods with side effects are called in the form of an imperative verb: `list.sort()`.

Similar non-mutating methods that return new values must be called using past participle: `let newList = oldList.sorted()`. Or present participle: `let newList = oldList.sortingLexicographically()`.

Parameter names follow rules for variable names. An engineer shall make use of Swift syntax possibilities to make the full declaration a human-readable phrase. For example, `func insert(_ element: Element, at index: Int)` is nicely read as `insert(someElement, at: someIndex)` at the call site.

Factory method names start with the word "make". Both factory methods and initializers have their parameters as a list of items, they are not required to form phrases.

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

UIKit's `UIControl` actions are called with the control's name in the beginning and the "action" word in the end:

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

Basically, types representing pure values which could be fairly interchanged by values themselves should be `struct`s, otherwise (e.g., objects containing state or having a lifecycle) – `class`s. Other considerations for choosing over `struct`s and `class`es – expected semantics, memory management, etc., are not the topic of this document.

Properties are expected to have a constant time complexity. If one doesn't, it's changed to a method.

Optional booleans and collections are avoided, because they bring degraded states: What's the difference between an empty array and a `nil` array within the same context?

Free functions are usually a design flaw. An engineer shall double-check, whether free functions really shouldn't correspond to some type.

Methods that perform an action don't return the object of the action (e.g., a presenting view controller method shall not return the presented view controller).

**Do**:

```swift
func presentErrorAlert()
```

**Don't**:

```swift
func presentErrorAlert() -> UIAlertController
```

However, if the return value might be useful in some specific situations, it doesn't force one to use it (the `@discardableResult` annotation serves this purpose). Though, such return values are not encouraged. An engineer shall follow the principle of [the command-query separation](https://en.wikipedia.org/wiki/Command–query_separation).

Abbreviations and acronyms (except for the common ones) are avoided. (Possible exceptions are local variables within implementation blocks of code.)

**Do**: `let currentValue = 1`

**Don't**: `let curValue = 1` (Though, it's acceptable in a very limited scope, like a method implementation).

<h4 id="encapsulation">Encapsulation</h4>

Any class declaration gains from adding the `final` modifier, an engineer adds it by default and remove in case of a necessity. (An engineer shall prefer composition over inheritance.)

`fileprivate` access modifier is usually a code smell. An engineer shall re-consider an alternative design approach (e.g., nested types).

The default access modifier, namely `internal`, is not specified explicitly (as well as any other default, in general).

`class` type members are rarely useful because of discouraged use of inheritance, especially for static members. An engineer must be aware of the use-cases of `class` and `static` modifiers (confusing them is a common mistake.)

Generally, encapsulation is honored in any way. E.g., `@IBOutlet` properties and `@IBAction` methods are always `private`. Implementation details are hidden behind meaningful API.

[Return to Table of Contents](#table-of-contents)

<h3 id="implementation">Implementation</h3>

An engineer makes use of type inference and the current namespace. The compiler notifies them when a type should be stated explicitly. However, there might be occasions when specifying a type explicitly makes code more effective.

Similar rules are applied to nested types and enum cases. The shorthand notation starting with a dot is preferred whenever is possible.

An engineer makes use of lazy initialization of the properties that aren't immediately needed. They also prefer complex lazily initialized properties for subviews of UIKit's views and view controllers, especially over configuring them within view controller's lifecycle methods (a common mistake is to write big and awkward `viewDidLoad()` implementations consisting of the full view controller's initial configuration.)

Conditional code checks "the golden path" (the most probable option) first. Inverted checks (like `!incorrect`) are discouraged because of poor readability. When both rules appear to be mutually exclusive, alternatives shall be considered.

**Initialization**

If the initial or constant value of a property doesn't depend on the initializer's parameters, a default value is preferred over setting it within the initialization code.

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

An engineer avoids force-unwraps. Basically, if an object is optional, that's because it really might be `nil` in certain conditions. If an engineer is 100% sure (which is usually impossible) that the optional value can't be `nil`, it's better to state their point of view explicitly. For instance, by adding some kind of assertion inside a `guard` statement.

A possibly justified reason to have a force-unwrapped variable is late initialization. However, the latter may also be error-prone and should be avoided.

Another acceptable reason to have a force-unwrap is an optional API which returns `nil` only if there's a programming error (e.g., creating an URL from `String` in Foundation : `let url = URL(string: "https://www.apple.com")!`). However, an engineer should not rely on third-party APIs' implementation details since they might change any moment and without notice. 

In testing code, force-unwraps are more welcome because if they fail, the test fails too, and this is desired.

<h4 id="state-and-side-effects">State and Side Effects</h4>

Clean functions are preferred over state-changing ones. Unidirectional flow is preferred over side effects. It makes code less error-prone, improves readability and testability.

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

_I'm aware that it's not the canonical, functional, definition of the term "pure function". Here I use it in the OOP context, for a basic understanding of the idea._

<h4 id="testing">Testing</h4>

An engineer tests interfaces, not implementation – by calling public APIs and asserting the expected output against an input. 

Testing purposes don't intervene in the principles of encapsulation. If anything is wanted to be overridden or substantiated in testing code, protocols and their mock or stub implementations are always to the rescue.

[Return to Table of Contents](#table-of-contents)

<h3 id="formatting">Formatting</h3>

Multiple variables declarations on a single line using the single `var`/`let` keywords (like `var x: Int, y: Int`) are restricted. Readability shall not be sacrificed for brevity.

Custom horizontal alignment isn't used because it welcomes further maintenance problems:

**Don't**:

```swift
let top:    CGPoint
let middle: CGPoint
let bottom: CGPoint
```

**Type aliases**

`typealias` declaration is used only for the sake of brevity when it doesn't prevent clarity.

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

A method declaration is placed on a single line if it can fit most display screen widths without a carry-over. Otherwise, each parameter is placed on its own line and matches the beginning of the previous one. Return type carries on to the last parameter's line.

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

A bottomline: here's the prioritized order, in which the arguments are formatted, depending on the available space:

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

Operator functions have a space between the implemented operator's name and the left parens.

**Do**:

```swift
static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
```

**Line length**

Java Code Conventions can be used for deducing a maximum characters-per-line value. Or common sense, as an option – from 80 to 120 symbols per line.

**Annotations**

Attributes starting with a `@` symbol shall be on separate lines before the corresponding main declarations.

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

Long expressions are broken into several parts on different lines so that the symbol that connects two parts of expression starts the new line. Each new line is indented with one additional indentation level. Having a special character starting a new line avoids creating the illusion that a new line is a new statement.

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

Code within the scope (e.g., method's implementation) is separated into logical blocks by empty lines. If all blocks are one line, no separation is used. `return` statement is usually separated, the only exception is if there's only two lines overall.

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

If the implementation consists only of a return statement, the `return` keyword is omitted:

**Do**:

```swift
func getViewController() -> UIViewController {
    vc
}
```

<h4 id="closures">Closures</h4>

Trailing closure syntax is preferred anywhere possible. Therefore, methods with closures as arguments aren't overloaded differing only by the name of the trailing closures – that leads to ambiguity on the call site.

Empty parentheses are not used on the call site for the methods with the only closure as an argument.

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

The functional style is preferable whenever possible: explicit argument names are omitted (placeholders like `$0` are used), one line closure content scope is placed on the same line with braces (with respect to the line width limits, though).

**Don't**:

```swift
requestMoreMoney { money in
    spend(money)
}
```

Chained functional-style calls, if don't fit a single line have line breaks before a dot. Even the first call of the chain is not left on the first line. Each new line of this call has exactly one extra level of indentation.

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

One indentation level is also used after parentheses.

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

Braces are open on the same line with declaration and closed on a separate line. No empty lines are left after opening brace and before the closing one. (The only exception is functional-style calls or, say, property observers which syntactically might be considered as functional statements. See the Methods and Closures sections.)

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

Multiple conditions of a single `if`-statement are placed on separate lines. Using commas instead of logic "and" (`&&`) is preferred.

**Do**:

```swift
if password.isCorrect,
    user.exist {
    // ...
```

**Don't**:

```swift
if password.isCorrect, user.exist {
    // ...
```

```swift
if password.isCorrect 
    && user.exist {
    // ...
```

If multiple paths exist, the most likely one goes first. Early returns are preferred over multi-level conditional paths.

**Do**:

```swift
if currency.supported {
    router.proceedToWithdrawal()
    return
}
if country.isAlly {
    router.proceedToAgreement()
    return
}
createAgreement()
```

**Don't**:

```swift
if currency.supported {
    return withdrawnAmount
} else if country.isAlly {
    return withdrawnAmount / 2
} else {
    return 0
}
```

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

Multiple and complex logical conditions are distinguished with braces. Distinguished conditions are placed on their own lines, with corresponding indentation:
 
`let goodToGo = ((firstNumber > 0)
        || (firstNumber < -10)) // Two levels of indentation, because it's a sub-condition of the first part of the main condition expression.
    && isFirstTry`. // One level of indentation, because it's the second part of the main condition expression.

Unnecessary parentheses within `if`-statement conditions are avoided:

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

If the control flow's main token is followed by left parentheses, exactly one standard space is inserted between them.

The ternary operator `?`-`:` shall not be used where the single `if`-check is sufficient, because although it can save lines, it makes the intention unclear and spawns extra entities (empty tuples or functions).

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

Ternary operator expressions are placed on three lines, with line breaks before the first and the second symbols of the operator. 

The `for`-`where` clause is preferred over the `if`-statements nested inside loops.

Clearly named, easy-to-understand local constants are preferred over inlined conditions:

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

One level of indentation is used inside a `switch`'s parentheses and for `case` implementations.

All statements inside the cases of a `switch` statement start on separate lines.

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

`default` cases are not used (unless it's an imported, non `NS_CLOSED_ENUM`, Objective-C enumeration) because they may lead to erroneous states if new cases are added.

<h4 id="literals">Literals</h4>

**Arrays**

Array literals shall not contain spaces after the left square bracket and before the right one. The included items shall be listed one below another, aligned at the same level of indentation. The first element shall be on the line right next after the declaration. The closing bracket shall go on the next line after the last item. However, if items are short and their sequence can be read easily (e.g., integer literals) it's acceptable to have them all on the one line.

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

The option with braces on separate lines is used when elements don't fit the line width:

**Do**:

```swift
let airVehicles = [
    assumeThisNameDoesNotFitTheLineWidth,
    airLiner
]
```

The trailing comma after the last element shall not be used.

**Strings**

Multi-line strings shall be preferred over concatenation.

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

Long numbers shall have underscores separating groups of digits: `1_000_000_000`

Floating-point numbers all shall carry an explicit decimal fraction, even when it's zero. Breaking this rule leads to ambiguity when decimal number variables are assigned with new values: `let interval: TimeInterval = 2.0`

**Don't**:

```swift
let coordinate: CGFloat = 2.1
// ...
coordinate = 2 // Without looking at the declaration it's impossible to know that it's a floating point number.
```

<h4 id="constants">Constants</h4>

Global constants shall be avoided.

The constants within a type declaration shall be grouped logically into private case-less `enum`s as their static properties. Using case-less `enum`s instead of structures or classes prevents unwanted initialization of the containing entity, without adding an explicit private empty initializer.

<h4 id="colons">Colons</h4>

Colons always have no space on the left and one space on the right. Exceptions are ternary operators (`?`-`:`) and dictionaries.

**Do**: `let scoreTable: [String : Int] = [:]`

**Don't**s (three of them in a row – spot them all): `let scoreTable : [String: Int] = [ : ]`

The ampersand in composition statements (`Codable & Encodable`) is surrounded by spaces too.

**Operators**

All operators (including `=`, but excluding the already discussed `:`) have exactly one standard space before and after.

**Do**: `let a = b + c`

**Don't**: `if (b+c) == 1 {`

<h4 id="macros">Pre-processor Directives</h4>

Any macros shall not be indented, the surrounded code shall be formatted as if the macro doesn't exist.

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

Semicolons are not used. The only case in Swift when they are necessary is multiple statements on a single line, which are strongly discouraged.

Only the Latin alphabet and numbers are used in the source code (no Cyrillic alphabet, hieroglyphs, emojis, etc.). Although the Swift programming language supports any UTF symbols, they are hard to type using some of the various localized keyboards and encourage error-prone copying-and-pasting. 

Unused code (including imports and resources) shall be always removed, as well as empty or inherited method implementations.

Although empty blocks of code indicate design flaws, if they are unavoidable, they are filled with a comment explaining why it's empty:

**Do**:

```swift
func buttonAction() {
    // The button is inactive in this state and don't need an action.
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
