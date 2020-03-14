# Ultimate Swift Code Style Guidelines

The main purpose of these guidelines is to determine specific rules following of which may make one's code having a unified style – which improves readability and maintanability, hence less fallability. Most of other existing similar guidelines don't provide exhaustive requirements on any code style aspect, or aren't sufficiently accurate, or have their own purposes which don't correlate with the general software development. This document is a try to create one that does.

An engineer is free to use these guidelines changing any rules according to their wish. However, if any rule is broken it must be broken the same way across the whole project. I.e., any broken rule shall be replaced by another, instead of carelessly neglecting it.

One handy way to base own guidelines on these ones is to fork the repository and change any rule. However, if an engineer feels that something is completely wrong, they are free to fork and create a pull-request for initiating a discussion.

<h2 id="table-of-contents">Table of Сontents</h2>

## 1. [Guidelines](#guidelines)

### 1.1. [Project](#project)

#### 1.1.1. [Files and Directories](#files-and-directories)

#### 1.1.2. [CVS](#cvs)

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

### 1.6. [Implementaion](#implementation)

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

### 1.8. [Miscellaneous](#miscellaneous)

## 2. [Links](#links)

<h2 id="guidelines">Guidelines</h2>

<h3 id="project">Project</h3>

Projects don't tolerate any warnings.

[Return to Table of Contents](#table-of-contents)

<h4 id="files-and-directories">Files and Directories</h4>

All folders and files in a directory or a project section are sorted alphabetically, first directories, then files.

File sorting doesn't consider files' extensions. However, an engineer is strongly encouraged to name files to have files which are close to each other semantically remaining closer to each other in the list.

Files are called after their containing types, or the primary one if multiple type declarations are contained.

If the primary content of the file is a type extension, the file shall be named after this type concatenating a `+` sign and conformance protocol name. E.g., `String+ConvertibleToNumber.swift`.

[Return to Table of Contents](#table-of-contents)

<h4 id="cvs">CVS</h4>

_Not going deeply into the topic, since each CVS has its own guidelines._

Every commit is atomic and present a meaningful and working state of the project. Checking out at any commit results in a working software. Considering these two rules, an engineer shall think thoroughly about implementation phases, or refactoring steps.

Commit name is formulated in the imperative case ("Do the thing"). Comprehensive details are optional and separated by a blank line if present.

Any temporary code, anything is committed. Otherwise it will live while the repository does.

[Return to Table of Contents](#table-of-contents)

<h3 id="file-structure">File Structure</h3>

File is divided into sections by `// MARK:` comments. Root declarations (e.g. two classes on the same level) are separated by `// MARK: -`. Declarations within the same group of a type are gathered together in sections, like `// MARK: - Methods`. Multiple declarations within a group – like `// MARK: Private methods`. This type of comments (as well as any other) is kept close to grammatically correct English.

**Don't:**
```swift
//MARK:

//MARK :

// MARK: -Methods
```

Examples of file structure: 

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

final class GatewayStandartService: GatewayService {

    enum GatewayType {

        case demo
        case stage

    }

    // MARK: - Properties

    // MARK: GatewayService protocol properties

    weak var delegate: GatewayServiceDelegate?
    var active: Bool {
        return server.active
    }
    var data: [GatewayDataObject] {
        return server.data
    }

    // MARK: Private properties

    private static let gatewayStandardAddress: URL = URL(string: "https://standard.gateway.edu/")!
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

extension GatewayStandartService: DataServerDelegate {

    // MARK: - Methods

    // MARK: GatewayStandartService protocol methods

    func connectionEstablished() {
        notifyDelegate()
    }

    func dataUpdated() {
        notifyDelegate()
    }

    // MARK: Private methods

    private func notifyDelegate() {
        delegete?.dataUpdated()
    }

}
```

**Don't**:

```swift
import Foundation // FIXME: Wrong inports order
import GatewayFoundation
import UIKit // FIXME: Unused import
import Darwin // FIXME: Excess import, Foundation includes Darwin.

// FIXME: Extra empty line
protocol GatewayServiceDelegate { // FIXME: Absent section dividers.

    func dataUpdated() // FIXME: Broken identation

}
// FIXME: Absent file structural divider.
protocol GatewayService {

    // MARK: - Properties 

    var active: Bool { get }
    var data: [GatewayDataObject] { get }
    var delegate: GatewayServiceDelegate? { get set } // FIXME: Less strict accessable properties should go first.

    // MARK: - Methods

    func connect()
    func disconnect()
} // FIXME: Absent empty line before type declaration closing brace. 

// MARK: -

final class GatewayStandartService: GatewayService, DataServerDelegate { // FIXME: Additional conformances should be declared in extensions.

    // MARK: - Properties

    // MARK: GatewayService protocol properties

    private static let gatewayStandardAddress: URL = URL(string: "https://standard.gateway.edu/")! // FIXME: Private and "public" members are mixed.
    private let serviceType: GatewayType
    weak var delegate: GatewayServiceDelegate?
    var data: [GatewayDataObject] { return server.data } // FIXME: Braces (non-functional) content must go on separate lines.
    var active: Bool { // FIXME: Properties of the same access level should be sorted alphabetically.
        return server.active
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
        // FIXME: Empty lines in the beginning and the end of the method. 
        server.breakConnection()

    }

    // MARK: GatewayStandartService protocol methods

    func connectionEstablished() {
        notifyDelegate()
    }

    func dataUpdated() {
        notifyDelegate()
    }

    // MARK: Private methods

    private func notifyDelegate() {
        delegete?.dataUpdated()
    }

    enum GatewayType { // FIXME: Nested types accessible from the outside of the type should go first. 
        case demo
        case stage
    } // FIXME: No both opening and closing braces.

}
```

File length doesn't have a specific line number limitation, since basically, code shall respect contemporary software development principles (mainly, S.O.L.I.D., and also complemented by their small siblings – D.R.Y., K.I.S.S. and Y.A.G.N.I.). Following the principles won't let an engineer to create an unacceptably long files. However, if an engineer isn't experienced and doesn't have an experienced reviewer, the number of 200 lines (including all formatting and comments) could be used for guidance. 

[Return to Table of Contents](#table-of-contents)

<h4 id="imports">Imports</h4>

All imports go in the very beginning of the file and sorted lexicographically. Empty lines between imports are not used.

More specific imports (like `Darwin`) are preferrable over less specific (like `Foundation`) to keep name space cleaner and probably, the resulting build thinner.

More specific imports (like `Foundation`) are not imported explicitly if less specific ones (like `UIKit`) are also used.

[Return to Table of Contents](#table-of-contents)

<h4 id="members-order">Members' Order</h4>

Properties within a subgroup (as well as enum cases) must be ordered, firstly, by its access modifier, then by the class/instance membership type, then lexicographically.

Logical sorting is acceptable, but not encouraged and may be used only if all declarations can be sorted logically. Partially logical and partially lexicographical ordering is avoided.

All type declarations shall have an empty line after a left brace and before the right one, placing content as if it's in between of surrounding bounds. At the same time, more that one empty line usage is prohibited.

Root declarations start from the beginning of the line. Each level of nesting adds a step of indent, one indent is either four spaces or one tabulation symbol.

Public methods shall be sorted in a logical order. It's always possible to decide on one for methods. For example, by remembering the type's lifecycle and methods' calling order.

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

Private methods are sorted by their first mention (earlier mentions go first). Mixing public and private APIs shall be strictly avoided. However, the latter is discussible. An alternative could be to place methods in order of their usage and closer to the first calling. I.e., "as an engineer would read" the source file in order to understand the flow.

[Return to Table of Contents](#table-of-contents)

<h4 id="protocol-conformances">Protocol Conformances</h4>

Protocol conformances are added within separate extensions unless it's the main point of type existance.

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

Protocol extensions go after the main declaration and are separated by a ```// MARK: -``` comment with the protocol name. It results in a nice "table of contents" in the file navigator window of an IDE.  

**Do**:

```swift
struct SomeType {

    // Main implementation goes here.

}

// MARK: - DifferentProtocol

extension SomeType: DifferentProtocol {

    // MARK: - Methods

    // MARK: DifferentProtocol protocol methods

    // Protocol conformance implementation goes here.

}
```

[Return to Table of Contents](#table-of-contents)

<h3 id="explicit-documentation">Explicit Documentation</h3>

Each non-private API have [HeaderDoc](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/HeaderDoc/intro/intro.html) documentation even the API is well self-documented. The main assignment of having exhaustive documentation of public APIs is to be visible from different code base places through IDE's help features, such as quick help popovers.

Private APIs are well self-documented and have no explicit documentation in order not to distract from reading implementation details.

Indicating method's time complexity in documentation usually is a good idea.

One line documentation is denoted as `\\\`:

```swift
\\\ Summary.
```

Multi-line documentation is put between `/**` and `*/` each of which must be on separate lines. `*` prefixes on new lines are not used.

Basic structure of multi-line descriptions:

**Do:**

```swift
/**
Summary.

Detailed description.

- Parameters:
    - parameter1: Parameter description.
    - parameter2: Parameter description.

- Returns: Return value description.

- Throws: Exception description.
*/
```

**Don't:**

```swift
/// Summary.
/// - parameter parameter1: Parameter description.
/// - parameter parameter2: Parameter description.
/// - returns: Return value description.
```

This recommended style also may appear familiar because of similarity to JavaDoc (which is not an end in itself, but adds some comfort for experienced engineers).

All but summary are optional. The order shall be:
- Summary 
- Discussion
- `Parameter`/`Parameters`
- `Returns`
- `Throws` 

If the method has parameters they are necessary to be documented. Single parameter must be documented as `- Parameter: Description.` Multiple parameters must be documented by help of `- Parameters:` syntax. If the method have functions as parameters, their arguments must have labels and be documented, as well.

If the method's return value type is not `Void` it's documented.

If the method throws an exception it's documented.

Two HeaderDoc parts are divided by a single blank line. However, if there's no description and a method contains not more than one parameter, blank lines are be omitted:

```swift
/**
Summary.
- Parameter parameter: Parameter description.
- Returns: Return value description.
- Throws: Exception description.
*/
```

Links, code snippets etc. must make use of [Apple Markup](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/) syntax.

All text is written in American English, gramatically correct (including punctuation, such as periods at the end of each sentence). Every phrase (even not being a complete sentence) is ended by a period.

**Do**: `func summarizeFavorites()`

**Don't**: `func summariseFavourites()`

[Return to Table of Contents](#table-of-contents)

<h4 id="comments">Comments</h4>

Self-documented code is preferred over comments.

If a commented code is being changed, the comments are either updated, or deleted. 

One line comments start with `\\`  (although `\\\` makes comments look fancier, it's reserved for HeaderDoc).

Multi-line comments start with `\*` and end with `*\`. Those are placed on separate lines. No blank lines after symbols denoting the beginning of a comment and before ones at the end.

Comments, both single- and multi-line, are placed on the line above the commented code and separated from a preceding code by a blank line. The latter is not relevant if a comment is the first line of the scope. Also, it's acceptable to place one-line comments at the end of the commented line of code, separating it by a space, if and only if the comment doesn't make this line too long. 

[Return to Table of Contents](#table-of-contents)

<h3 id="naming">Naming</h3>

All declarations should be self-explanatory and consist of American English words and form correct human-readable phrases when readed aloud (i.e., not wording syntax symbols, such as function arguments brackets).

Everything but types is `lowerCamelCase`. The types are `UpperCamelCase` (a.k.a. `PascalCase`).

Abbreviations don't have different cases within and are cased up or down according to the rule above (e.g. `let asciiTable: [String: String]`, `class SMTPServer`). A common mistake to use `Id` instead of `ID` is also avoided.

No underscores, even at the beginning of property names, for making it recognizable, are used. No Hungarian notation for constants (e.g., leading "k"), also.

In most cases variables have noun phrase names: `let number: Int`. The most significant exception are Booleans which are expected to have assertive names (e.g., `let necessary: Bool`).

[Return to Table of Contents](#table-of-contents)

<h4 id="types">Types</h4>

Classes and structures are named using noun phrases. As well as protocols describing what an object is. Protocols which add abilities are named descriptively (e.g., `Sortable`). Generalization protocols can have the word `Protocol` at the end of their name:

```swift
protocol AnimatedViewProtocol {
    // ...
}

final class AnimatedView: UIView, AnimatedViewProtocol {
    // ...
}
```

Types implementing design patterns are usually named with the pattern name at the end (e.g., `ViewBuilder`, `DisplayingStrategy`).

No prefixes may be used (e.g., just `PriceCalculator` instead of `XYZPriceCalculator`), since there's no necessity for that in Swift (other than to maintain consistency with Objective-C libraries and components).

[Return to Table of Contents](#table-of-contents)

<h4 id="methods">Methods</h4>

Methods without side-effects have names in the form of noun phrases: `let size = view.originalSize()`.

Methods with side effects are called in the form of an imperative verb: `list.sort()`.

Similar non-mutating methods that return new values must be called using past participle: `let newList = oldList.sorted()`. Or present participle: `let newList = oldList.sortingLexicographically()`.

Argument names follow rules for variable names. An engineer makes use of Swift syntax possibilities, for example: `func insert(_ element: Element, at index: Int)`.

Factory method names start with the word "make". Both factory methods and initializers have their arguments as a list, conventionally it's not necessary for them to form phrases.

**Do**:

```swift
init(name: String, id: Int)

func makeView(position: CGPoint, size: CGSize) -> UIView
```

It's very common to force engineers to put an object of delegation as a first argument of delegation methods. This is not strictly necessary and is used only in cases when it is really possible and makes sense for a type to be a delegate of multiple objects of the same type.

**Do**:

```swift
protocol ScreenPresenter {

    // MARK: - Methods
    
    func buttonTapped()
    
}
```

**Don't** (most probably):

```swift
protocol ScreenPresenter {

    // MARK: - Methods
    
    func buttonTapped(_ screen: UIViewController, button: UIButton)
    
}
```

UIKit's `UIControl` actions are called with the control's name in the beginning and the "action" word in the end:

**Do**:

```swift
@objc
private func nextButtonAction(_ sender: UIButton) { // ...
```

The sender argument is optional since it stays unused very often nowadays.

**Don't**:

```swift
@objc
private func onNextButtonTapped(_ sender: UIButton) { // ...
```

The latter naming convention is confusing because makes you think of delegation. However, actions may be thought of as a kind of delegation, and a naming convention like that might be considered acceptable.

[Return to Table of Contents](#table-of-contents)

<h3 id="api">API</h3>

Basically, types representing pure values which could be fairly interchanged by values themselves should be `struct`, otherwise (e.g., objects containing state of having a lifecycle) – `class`. Other considerations for choosing over `struct`s and `class`es – expected semantics, memory management etc., are not of the topic of this document.

Properties are expected to have a constant time complexity. If one doesn't it's changed to a method.

Optional Booleans and collections are avoided because they bring degraded states: for example, what's the difference between an empty array and a `nil` array within the same context?

Free functions are usually a design flow. If one exists, an engineer double checks whether it really shouldn't correspond to any type.

Methods which do some actions don't return objects of these actions (e.g., a presenting view controller method shall not return the presented view controller).

**Do**:

```swift
func presentErrorAlert()
```

**Don't**:

```swift
func presentErrorAlert() -> UIAlertViewController
```

However, if the return value might be useful in some specific situations, it doesn't force one to use it (`@discardableResult` annotation serves this purpose).  

Abbreviations (except for the common ones) and shortenings are avoided. (Possible exceptions are local variables within implementation blocks of code.)

**Do**: `let currentValue = 1`

**Don't**: `let curValue = 1`

[Return to Table of Contents](#table-of-contents)

<h4 id="encapsulation">Encapsulation</h4>

Any class declaration gains of adding `final` modifier, an egnineer adds it as a default and remove in case of a real necessity. (Experienced engineers prefer composition over inheritance.)

`fileprivate` access modifier is usually a code smell. An engineer must re-consider an alternative design approach (e.g., nested types).

The default access modifier, namely `internal`, is not specified explicitly (as well as any other default, generally).

`class` type members are rarely useful because of discouraging to use inheritance, especially for the static members. An engineer must be aware of use cases of `class` and `static` modifiers, confusing them is a common mistake.

Generally, encapsulation is honored in any way. E.g. `@IBOutlet` properties and `@IBAction` methods are always private. Implementation details are hidden behind meaningful API.

[Return to Table of Contents](#table-of-contents)

<h3 id="implementation">Implementation</h3>

An engineer makes use of the type inference and the current name space. Compiler notifies them when the explicit type is necessary. However, there might be occasions when explicit types make code a little bit, though significantly, more effective.

Similar rules are applied for nested types and enum cases. The shorthand notation starting with dot is preferred whenever is possible.

An engineer makes use of lazy instantiation for properties which aren't immediately needed. They also prefer complex lazily instantiated properties for subviews of UIKit's views and view controllers, especially over configuring them within view controller's lifecycle methods implementation (a common mistake is to write big and awkward `viewDidLoad()` implementations consisting of the full view controller's initial configuration).

Conditional code checks "the golden path" (the most probable option) first. Inverted checks (like `!incorrect`) are discouraged because of poor readability. When both rules appear to be mutually exclusive, an alternative way to write down the flow must be considered.

**Initialization**

If initial or constant value of a property doesn't depend on the initializer's parameters, a default value is preferred over setting it within the initialization code.

`.init()` is not used for initialization:

**Do**:

```swift
let color = UIColor(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
```

**Don't**:

```swift
let color: UIColor = .init(displayP3Red: 1.0, green: 0.0, blue: 0.0, alpha: 1.0)
```

[Return to Table of Contents](#table-of-contents)

<h4 id="implementation-methods">Methods</h4>

Any scope contains only elements of the same level of abstraction (i.e., variables and related direct calculations, or method calls of the same layer).

**Do**:
```
func viewDidLoad() {
    super.viewDidLoad()
    presenter.viewLoaded()
}
```

**Don't**:
```
func viewDidLoad() {
    super.viewDidLoad()

    let button = UIButton()
    button.addTarget(self, action: #selector(buttonAction), event: .touchUpInside)
    layoutButton()

    view.backgroundColor = .black
}
```

Methods, as well as any code part, follow the Big-S. (Single Responsibility Principle) of S.O.L.I.D. principles. Besides the maintainability, that doesn't let methods grow long and difficult to read. However, as for length, there might be exceptions. For example, complex drawing methods that can't be broken up into several small ones (or breaking up would make the code less readable and more complex than before).

[Return to Table of Contents](#table-of-contents)

<h4 id="optionality">Optionality</h4>

An engineer avoids force-unwraps. Basically, if any object is optional that's because it really might be `nil` in certain conditions. If an engineer is 100% sure (which is, I'm 100% sure, impossible) that the optional value can't be `nil`, it's better to state their point of view explicitly. For instance, by adding some kind of assertion inside `guard let`-`else` statement.

A possibly justified reason to have a force-unwrapped variable is late initialization. However, the latter may also be error-prone and sometimes is avoided.

Another acceptable reason to have a force-unrwrap is an optional API which returns `nil` only if there's a programming error (e.g., creating a URL from `String` in Foundation library: `let url = URL(string: "https://www.apple.com")!` – that one just can't return `nil` unless there's a serious typo). However, an engineer doesn't rely on third-party APIs' implementation details since they might change any moment and without notice. 

In testing code force-unwraps are welcomed because if they fail, the test fails, too, and that is a desired behavior in this case.

[Return to Table of Contents](#table-of-contents)

<h4 id="state-and-side-effects">State and Side Effects</h4>

Clean functions are preferred, any necessary state elements are localized as locally as possible. Unidirectional flow is preferred over side effects. That makes the code less error-prone, improves readability and testability.

**Don't**:
```
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

```
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

In the latter example, `button` is initialized once, within the `viewDidLoad()` method. Once it's set, `configure(button:)` is called – a pure function that doesn't rely on any kind of state and has no side effects. That is, it always results the same way for the same arguments.

_I'm aware that it's not the canonical, functional, definition of the term "pure function". Here I use it in the OOP context, if I may, for a simple basic understanding of the idea._

[Return to Table of Contents](#table-of-contents)

<h4 id="testing">Testing</h4>

Basically, an engineer tests interfaces, not implementation details – by calling public APIs with some input data and asserting the expected outputs. 

The testing purposes don't intervene the principles of encapsulation. If anything is wanted to be overridden or substantiated in the test code, protocols and their mock or stub implementations are always to the rescue.

[Return to Table of Contents](#table-of-contents)

<h3 id="formatting">Formatting</h3>

Multiple variables declarations on a single line using the single `var`/`let` keywords (like `var x: Int, y: Int`) are restricted. Readability is not sacrificed for brevity.

Custom horizontal alignment isn't used because it welcomes further maintenance problems. E.g.:

**Don't**:

```swift
let top:    CGPoint
let middle: CGPoint
let bottom: CGPoint
```

**Type aliases**

`typealias` declaration is used only for the sake of brevity when it doesn't prevent of clarity.

**Do**:

```swift
typealias Task = (_ token: Sting) -> (_ result: Result)
func enqueue(task: Task)
```

**Don't**:

```swift
typealias T = (Int, Int) -> (String)
func process(task: T) -> String
```

[Return to Table of Contents](#table-of-contents)

<h4 id="formatting-methods">Methods</h4>

A method declaration is placed on a single line if it can fit most display screen widths without a carry-over. Otherwise each parameter is placed on its own line and match the beginning of the previous one. Return type carries on to the last parameter's line.

**Do**:

```swift
func fetchResults(from endpoint: URL,
                  transferringTo device: Device,
                  compressed: Bool,
                  completionHandler: (() -> ())?) –> [Data]
```

If any of the parameters don't fit on the screen, the option with parentheses on separate lines is acceptable.

**Do** (or rather an acceptable formatting option):

```swift
func fetchResults(
    from endpoint: URL = .remoteServerPath,
    transferringTo device: Device = .current,
    compressed: Bool = true,
    completionHandler: ((_ success: Bool) -> ())? = nil
                  ) –> [Data]
```

**Don't**:

```swift
func fetchResults(from endpoint: URL, transferringTo device: Device, 
                  compressed: Bool, completionHandler: (() -> ())?) –> [Data]
```

In the latter example parameters which go second on the lines are hard to notice.

**Don't**:

```swift
func fetchResults(from endpoint: URL, transferringTo device: Device, compressed: Bool, completionHandler: (() -> ())?) –> [Data]
```

The latter example has a string that might be too long for some displays when rendered within an IDE's code area.

**Custom operators**

Operator implementing functions have a standard space between the implemented operator's name and the left parens.

**Do**:

```swift
static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
```

**Line length**

Java Code Conventions can be used for deducing a maximum symbols-per-line value. Or common sense, as an option – from 80 to 120 symbols per line.

**Annotations**

Attributes starting with a `@` symbol go onto a separate line before the declaration.

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

Long expressions are broken into several parts on different lines so that the symbol that connects two parts of an expression starts the new line. Each new line is indented with one additional indentation. Having a special character starting a new line avoids creating the illusion that a new line is a new statement.

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

Code within the scope (e.g., a method's implementation) is separated into logical blocks by empty lines. If all blocks are one line, no separation is used. `return` statement is separated, the only exception is if the implementation consists of two lines.

**Do**:

```swift
func makeOrderViewController(orderNumber: Int) -> UIViewController {
    guard let order = Order(orderNumber: Int) else {
        fatalError("Unexisting order.")
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

[Return to Table of Contents](#table-of-contents)

<h4 id="closures">Closures</h4>

Trailing closure syntax is preferable anywhere possible. Therefore, methods with closures as arguments aren't overloaded differing only by the name of trailing closures – that leads to ambiguity on the call site. Empty parentheses are not used on the call site for the methods with the only closure as an argument.

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

Functional style is preferable whenever possible: explicit argument names are omitted (placeholders like `$0` are used), one line closure content scope is placed on the same line with braces.

**Don't**:

```swift
requestMoreMoney { money in
    spend(money)
}
```

Chained functional-style calls, if don't fit the single line have line breaks before a dot. Each new line of this call has an additional level of indentation.

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

[Return to Table of Contents](#table-of-contents)

<h4 id="control-flow">Control Flow</h4>

Braces are open on the same line with declaration and closed on a separate line. No empty line is left after open and before close. (The only exception is functional-style calls or, say, property observers which syntactically might be considered as functional statements. See Methods and Closures sections.)

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

If another style is preferred, braces position are never in mixed style.

Multiple conditions of a single `if`-statement are placed on separate lines. Using commas instead of logic "and" (`&&`) is preferred.

**Do**:

```swift
if password.isCorrect,
    user.exist { // ...
```

**Don't**:

```swift
if password.isCorrect, user.exist {  // ...
```

```swift
if password.isCorrect 
    && user.exist { // ...
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
launchRockets()
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

In any `guard`-statement, with either one or multiple conditions, `else` (and its left brace) goes on the same line after the last condition.

**Do**:

```swift
guard !array.isEmpty else { // ...
```

**Don't**:

```swift
guard !array.isEmpty 
    else { // ...
```

Multiple and complex logical conditions are distinguished with braces, like: `let goodToGo = (firstNumber > 0 || firstNumber < -10) && isFirstTry`.

Unnecessary paretheses within `if`-statement conditions are avoided:

**Do**:

```swift
if happens { // ...
```

**Don't**:

```swift
if (number != 0) { // ...
```

If control flow's main token is followed by a left parens, exactly one standard whitespace is inserted between them.

The ternary operator `?`-`:` is not used where the single `if`-check is sufficient because although it can save lines, it makes the intention unclear and spawns extra entities (empty tuples or functions).

**Do**:

```swift
if error == nil {
    completion()
}
```

**Don't**:

```swift
error == nil ? completion() : ()
```

The `for`-`where` clause is preferred over the `if`-statements nested inside loops.

Clearly named easy-to-understand local constants are preferred over inlined conditions:

**Do**:

```swift
let withinSolarSystem = 2 * 2 == 4
if withinSolarSystem { // ...
```

**Don't**:

```swift
if 2 * 2 == 4 { // ...
```

**Switch Statements**

All statements inside the cases of a `switch` statement start on separate lines.

**Do**:

```swift
switch direction {
case .left:
    turnLeft()
case .right:
    turnRight():
case .straight:
    break
}
```

**Don't**:

```swift
switch direction {
case .left: turnLeft()
case .right: turnRight():
case .straight: break
}
```

`default` cases are not used (unless it's an imported Objective-C enumeration) because they may lead to erroneous states if new cases are added. 

[Return to Table of Contents](#table-of-contents)

<h4 id="literals">Literals</h4>

**Arrays**

Array literals don't contain spaces after the left square bracket and before the right one. The included items are one below another, being aligned by their name's first letter/symbol. The first element is on the declaration's line. The right square bracket goes on the same line with the last item. If items are rather short and can be read easily (e.g., integer literals) it's acceptable to have them all on the one line.

**Do**:

```swift
var numbers = [1, 2, 3]

let airVehicles = [helicopter,
                   airLiner,
                   carrierRocket,
                   wings]
```

**Don't**:
```swift
var numbers = [
    1, 
    2, 
    3
              ]
              
let airVehicles = [helicopter, airLiner, carrierRocket, wings]
```

The trailing comma after the last element is not used.

**Strings**

Multi-line strings are preferred over concatenation.

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

**Numbers**

Long numbers have underscores separating groups of digits: `1_000_000_000`

Floating point numbers all carry an explicit decimal fraction, even when zero. Breaking this rule leads to ambiguity when decimal number variables are assigned with new values: `let interval: TimeInterval = 2.0`

**Don't**:

```swift
let coordinate: CGFloat = 2.1
// ...
coordinate = 2 // Without looking at the declaration it's impossible to know that it's not an integer.
```

[Return to Table of Contents](#table-of-contents)

<h4 id="constants">Constants</h4>

Global constants are avoided, leaving alone it's an obvious code smell.

The constants within a type declaration are grouped logically into private case-less `enum`s as their static properties. Using case-less `enum`s instead of structures or classes prevents the unwanted initialization of the containing entity, without adding the explicit private empty initializer.

[Return to Table of Contents](#table-of-contents)

<h4 id="colons">Colons</h4>

Colons always have no space on the left and one space on the right. Exceptions are: the ternary operator (`?`-`:`), the empty dictionary (`[:]`) and the selector syntax (e.g., `addTarget(_:action:)`).

**Do**: `let scoreTable: [String: Int] = [:]`

**Don't**s (three of them in a row – spot them all): `let scoreTable : [String:Int] = [ : ]`

The ampersand in composition statements (`Codable & Encodable`) is surrounded by spaces too.

**Operators**

All operators (including `=`) have exactly one standard whitespace before and one after.

**Do**: `let a = b + c`

**Don't**: `if b+c == 1 { \\...`

[Return to Table of Contents](#table-of-contents)

<h3 id="miscellaneous">Miscellaneous</h3>

No semicolons, never, at all. The only case in Swift when they are necessary is multiple statements on a single line, which is strongly discouraged.

Only latin alphabet and numbers are used in the source code. That means no Cyrillic alphabet, hieroglyphs, emojis, etc. Although the Swift programming language supports any UTF symbols, they are hard to reproduce using some of various localized keyboards and encourage error-prone copying-and-pasting. 

Unused code (including imports and resources) must be deleted. As well as empy and completely inherited implementations.

Although empty blocks of code indicate design flaws, if needed they are placed on one line and have exactly one standard space symbol between parenthesis. Empty methods are formatted as regular ones having one line inside which is a comment explaining why this method is empty. 

**Void**

`()`, `Void` and `(Void)` are syntactically interchangeable. However, the first one means an empty tuple which can be used, for example, as denoting a free function without parameters. The second one is used as an empty return from a free function. The latter is just a pointless usage of a possibility to put any statement between braces and is not used. 

**Do**: `let completionHandler: () -> Void`

**Don't**: `let completionHandler: () -> ()`

**Access modifiers**

Access modifiers of types, methods etc. shall go first.

**Do**: `public static func goPublic()`

**Don't**: `static public func goPublic()`

[Return to Table of Contents](#table-of-contents)

<h2 id="links">Links</h2>

[**Swift.org API Design Guidelines**](https://swift.org/documentation/api-design-guidelines/)

Non-neglectable official guides. Though, focusing mainly on APIs and naming.

[**Java Code Conventions**](https://oracle.com/technetwork/java/codeconvtoc-136057.html)

A good example of exhaustive code style for a programming language. Although it has many atavisms and old-fashioned rules, it's really surprising to come across such ugly formatting every here and there:

**Don't**:

```java
void layoutScreen() {
    layoutHeader()

    layoutFooter()
}
```

[**Steve McConnell – Code Complete: A Practical Handbook of Software Construction**](https://en.wikipedia.org/wiki/Code_Complete)

Impressive work guiding software development in general, paying a lot of reader's attention on code style and giving an exhaustive explanation on each rule. 
