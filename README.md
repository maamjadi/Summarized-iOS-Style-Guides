## Table of Contents

- [Table of Contents](#table-of-contents)
- [Goals](#goals)
- [Guiding Tenets](#guiding-tenets)
- [Xcode Formatting](#xcode-formatting)
    - [Why?](#why)
- [Naming](#naming)
  - [Use Type Inferred Context](#use-type-inferred-context)
- [Code Organization](#code-organization)
  - [Protocol Conformance](#protocol-conformance)
  - [Unused Code](#unused-code)
  - [Minimal Imports](#minimal-imports)
- [Spacing](#spacing)
- [Classes and Structures](#classes-and-structures)
  - [Which one to use?](#which-one-to-use)
  - [Example definition](#example-definition)
  - [Use of Self](#use-of-self)
  - [Computed Properties](#computed-properties)
  - [Return](#return)
- [Function Declarations](#function-declarations)
- [Function Calls](#function-calls)
- [Closure Expressions](#closure-expressions)
- [Types](#types)
  - [Constants](#constants)
  - [Static Methods and Variable Type Properties](#static-methods-and-variable-type-properties)
  - [Optionals](#optionals)
  - [Lazy Initialization](#lazy-initialization)
  - [Type Inference](#type-inference)
    - [Type Annotation for Empty Arrays and Dictionaries](#type-annotation-for-empty-arrays-and-dictionaries)
  - [Syntactic Sugar](#syntactic-sugar)
- [Functions vs Methods](#functions-vs-methods)
- [Memory Management](#memory-management)
  - [Extending object lifetime](#extending-object-lifetime)
- [Access Control](#access-control)
- [Control Flow](#control-flow)
  - [Ternary Operator](#ternary-operator)
- [Golden Path](#golden-path)
- [References](#references)


## Goals

Following this style guide should: 
* Make it easier to read and begin understanding unfamiliar code. 
* Make code easier to maintain. * Reduce simple programmer errors. 
* Reduce cognitive load while coding. 
* Keep discussions on diffs focused on the code's logic rather than its style. 

Note that brevity is not a primary goal. Code should be made more concise only if other good code qualities (such as readability, simplicity, and clarity) remain equal or are improved.


## Guiding Tenets 
* This guide is in addition to the official [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). These rules should not contradict that document. * These rules should not fight Xcode's <kbd>^</kbd> + <kbd>I</kbd> indentation behavior. 
* We strive to make every rule lintable:
  * If a rule changes the format of the code, it needs to be able to be reformatted automatically (either using [SwiftFormat](https://github.com/nicklockwood/SwiftFormat) or [SwiftLint](https://github.com/realm/SwiftLint) autocorrect).
  * For rules that don't directly change the format of the code, we should have a lint rule that throws a warning.
  * Exceptions to these rules should be rare and heavily justified.

## Xcode Formatting
You can enable the following settings in Xcode by running [similar script](resources/xcode_settings.bash), e.g. as part of a "Run Script" build phase._

* Each line should have a maximum column width of 130 characters.
#### Why?
  Due to larger screen sizes, we have opted to choose a page guide greater than 130.

  We currently only "strictly enforce" (lint / auto-format) a maximum column width of 130 characters to limit the cases where manual clean up is required for reformatted lines that fall slightly above the threshold.
* Use 2 spaces to indent lines.
* Trim trailing whitespace in all lines.


## Naming
Descriptive and consistent naming makes software easier to read and understand. Use the Swift naming conventions described in the [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/). Some key takeaways include:
- using `camelCase` (not `snake_case`)
- using `UpperCamelCase` for types and protocols, `lowerCamelCase` for everything else
- using names based on roles, not types
- naming methods for their side effects
  - verb methods follow the -ed, -ing rule for the non-mutating version
  - noun methods follow the formX rule for the mutating version
  - boolean types should read like assertions
  - protocols that describe _what something is_ should read as nouns
- generally avoiding abbreviations
- giving the same base name to methods that share the same meaning
- choosing good parameter names that serve as documentation
- preferring to name the first parameter instead of including its name in the method name, except as mentioned under Delegates
- taking advantage of default parameters


### Use Type Inferred Context
Use compiler inferred context to write shorter, clear code.  (Also see [Type Inference](#type-inference).)

**Preferred**:
```swift
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)
```

**Not Preferred**:
```swift
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

## Code Organization
Use extensions to organize your code into logical blocks of functionality. Each extension should be set off with a `// MARK: -` comment to keep things well-organized.

### Protocol Conformance
In particular, when adding protocol conformance to a model, prefer adding a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods.

**Preferred**:
```swift
class MyViewController: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
  // scroll view delegate methods
}
```

**Not Preferred**:
```swift
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

Since the compiler does not allow you to re-declare protocol conformance in a derived class, it is not always required to replicate the extension groups of the base class. This is especially true if the derived class is a terminal class and a small number of methods are being overridden. When to preserve the extension groups is left to the discretion of the author.

For UIKit view controllers, consider grouping lifecycle, custom accessors, and IBAction in separate class extensions.

### Unused Code
Unused (dead) code, including Xcode template code and placeholder comments should be removed. An exception is when your tutorial or book instructs the user to use the commented code.

Aspirational methods not directly associated with the tutorial whose implementation simply calls the superclass should also be removed. This includes any empty/unused UIApplicationDelegate methods.

**Preferred**:
```swift
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  return Database.contacts.count
}
```

**Not Preferred**:
```swift
override func didReceiveMemoryWarning() {
  super.didReceiveMemoryWarning()
  // Dispose of any resources that can be recreated.
}

override func numberOfSections(in tableView: UITableView) -> Int {
  // #warning Incomplete implementation, return the number of sections
  return 1
}

override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  // #warning Incomplete implementation, return the number of rows
  return Database.contacts.count
}

```

### Minimal Imports
Import only the modules a source file requires. For example, don't import `UIKit` when importing `Foundation` will suffice. Likewise, don't import `Foundation` if you must import `UIKit`.

**Preferred**:
```swift
import UIKit
var view: UIView
var deviceModels: [String]
```

**Preferred**:
```swift
import Foundation
var deviceModels: [String]
```

**Not Preferred**:
```swift
import UIKit
import Foundation
var view: UIView
var deviceModels: [String]
```

**Not Preferred**:
```swift
import UIKit
var deviceModels: [String]
```

## Spacing

* Indent using 2 spaces rather than tabs to conserve space and help prevent line wrapping. Be sure to set this preference in Xcode and in the Project settings as shown below:

![Xcode indent settings](uploads/ebb9c607d36d102e470f88b0b00823f1/indentation-2.png)

* Method braces and other braces (`if`/`else`/`switch`/`while` etc.) always open on the same line as the statement but close on a new line.
* Tip: You can re-indent by selecting some code (or **Command-A** to select all) and then **Control-I** (or **Editor ▸ Structure ▸ Re-Indent** in the menu). Some of the Xcode template code will have 4-space tabs hard coded, so this is a good way to fix that.

**Preferred**:
```swift
if user.isHappy {
  // Do something
} else {
  // Do something else
}
```

**Not Preferred**:
```swift
if user.isHappy
{
  // Do something
}
else {
  // Do something else
}
```

* There should be one blank line between methods and up to one blank line between type declarations to aid in visual clarity and organization. Whitespace within methods should separate functionality, but having too many sections in a method often means you should refactor into several methods.
* There should be no blank lines after an opening brace or before a closing brace.
* Closing parentheses should not appear on a line by themselves.

**Preferred**:
```swift
let user = try await getUser(for: userID,
                             on: connection)
```

**Not Preferred**:
```swift
let user = try await getUser(
  for: userID,
  on: connection
)
```

* Colons always have no space on the left and one space on the right. Exceptions are the ternary operator `? :`, empty dictionary `[:]` and `#selector` syntax `addTarget(_:action:)`.

**Preferred**:
```swift
class TestDatabase: Database {
  var data: [String: CGFloat] = ["A": 1.2, "B": 3.2]
}
```

**Not Preferred**:
```swift
class TestDatabase : Database {
  var data :[String:CGFloat] = ["A" : 1.2, "B":3.2]
}
```

* Long lines should be wrapped at around 70 characters. A hard limit is intentionally not specified.

* Avoid trailing whitespaces at the ends of lines.

* Add a single newline character at the end of each file.


## Classes and Structures

### Which one to use?
Remember, structs have [value semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_144). Use structs for things that do not have an identity. An array that contains [a, b, c] is really the same as another array that contains [a, b, c] and they are completely interchangeable. It doesn't matter whether you use the first array or the second, because they represent the exact same thing. That's why arrays are structs.

Classes have [reference semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_145). Use classes for things that do have an identity or a specific life cycle. You would model a person as a class because two person objects are two different things. Just because two people have the same name and birthdate, doesn't mean they are the same person. But the person's birthdate would be a struct because a date of 3 March 1950 is the same as any other date object for 3 March 1950. The date itself doesn't have an identity.

Sometimes, things should be structs but need to conform to `AnyObject` or are historically modeled as classes already (`NSDate`, `NSSet`). Try to follow these guidelines as closely as possible.

### Example definition
Here's an example of a well-styled class definition:

```swift
class Circle: Shape {
  var x: Int, y: Int
  var radius: Double
  var diameter: Double {
    get {
      radius * 2
    }
    set {
      radius = newValue / 2
    }
  }

  init(x: Int, y: Int, radius: Double) {
    self.x = x
    self.y = y
    self.radius = radius
  }

  convenience init(x: Int, y: Int, diameter: Double) {
    self.init(x: x, y: y, radius: diameter / 2)
  }

  override func area() -> Double {
    Double.pi * radius * radius
  }
}

extension Circle: CustomStringConvertible {
  var description: String {
    "center = \(centerString) area = \(area())"
  }
  private var centerString: String {
    "(\(x),\(y))"
  }
}
```

The example above demonstrates the following style guidelines:

 + Specify types for properties, variables, constants, argument declarations and other statements with a space after the colon but not before, e.g. `x: Int`, and `Circle: Shape`.
 + Define multiple variables and structures on a single line if they share a common purpose / context.
 + Indent getter and setter definitions and property observers.
 + Don't add modifiers such as `internal` when they're already the default. Similarly, don't repeat the access modifier when overriding a method.
 + Organize extra functionality (e.g. printing) in extensions.
 + Hide non-shared, implementation details such as `centerString` inside the extension using `private` access control.

### Use of Self
For conciseness, avoid using `self` since Swift does not require it to access an object's properties or invoke its methods.

Use self only when required by the compiler (in `@escaping` closures, or in initializers to disambiguate properties from arguments). In other words, if it compiles without `self` then omit it.


### Computed Properties
For conciseness, if a computed property is read-only, omit the get clause. The get clause is required only when a set clause is provided.

**Preferred**:
```swift
var diameter: Double { radius * 2 }
```

**Not Preferred**:
```swift
var diameter: Double {
  get {
    return radius * 2
  }
}
```

For getting values that don't require a prameter for their computation, instead of declaring functions, use computed properties.

**Preferred**:
```swift
var diameter: Double { radius * 2 }

func getDiameter(of view: UIView) -> Double {
    // code goes here
}
```

**Not Preferred**:
```swift
func getDiameter() -> Double { 
    // code goes here
}
```

### Return
From Swift 5.1, the `return` keyword can be omitted when declaring functions and computed properties that only contain a single expression. Therefore avoid the usage of `return` for these occasions.

While this new behavior might take a little while to get used to, it matches the way closures work, which does result in improved consistency between properties, methods, and closures:

**Preferred**:
```swift
extension MarkdownReader {
    var isAtStart: Bool { index == string.startIndex }
    var didReachEnd: Bool { index == string.endIndex }
    var currentCharacter: Character { string[index] }

    var computedCharacter: Character {
        // multiline code goes here
        return computedString[index]
    }
    
    func encode(character: String) -> String {
        character.encoded()
    }
}

// Here's how the above 'isAtStart' property might have been
// declared as a closure instead, prior to Swift 5.1:
let isAtStart: () -> Bool = { index == string.startIndex }
```

**Not Preferred**:
```swift
extension MarkdownReader {
    var isAtStart: Bool { 
        return index == string.startIndex
    }
    var didReachEnd: Bool { 
        return index == string.endIndex
    }
    var currentCharacter: Character { 
        return string[index]
    }
    
    func encode(character: String) -> String {
        return character.encoded()
    }
}

let isAtStart: () -> Bool = {
    return index == string.startIndex
}
```

For getting values that don't require a prameter for their computation, instead of declaring functions, use computed properties.

**Preferred**:
```swift
var diameter: Double { radius * 2 }

func getDiameter(of view: UIView) -> Double {
    // code goes here
}
```

**Not Preferred**:
```swift
func getDiameter() -> Double { 
    // code goes here
}
```

## Function Declarations

Keep short function declarations on one line including the opening brace:

```swift
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

For functions with long signatures, put each parameter on a new line and add an extra indent on subsequent lines:

```swift
func reticulateSplines(spline: [Double], 
                       adjustmentFactor: Double,
                       translateConstant: Int, 
                       comment: String) -> Bool {
  // reticulate code goes here
}
```

Don't use `(Void)` to represent the lack of an input; simply use `()`. Use `Void` instead of `()` for closure and function outputs.

**Preferred**:

```swift
func updateConstraints() -> Void {
  // magic happens here
}

typealias CompletionHandler = (result) -> Void
```

**Not Preferred**:

```swift
func updateConstraints() -> () {
  // magic happens here
}

typealias CompletionHandler = (result) -> ()
```


## Function Calls

Mirror the style of function declarations at call sites. Calls that fit on a single line should be written as such:

```swift
let success = reticulateSplines(splines)
```

If the call site must be wrapped, put each parameter on a new line, indented one additional level:

```swift
let success = reticulateSplines(spline: splines,
                                adjustmentFactor: 1.3,
                                translateConstant: 2,
                                comment: "normalize the display")
```


## Closure Expressions

Use trailing closure syntax only if there's a single closure expression parameter at the end of the argument list. Give the closure parameters descriptive names.

**Preferred**:
```swift
UIView.animate(withDuration: 1.0) {
  self.myView.alpha = 0
}

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}, completion: { finished in
  self.myView.removeFromSuperview()
})
```

**Not Preferred**:
```swift
UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
})

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}) { f in
  self.myView.removeFromSuperview()
}
```

For single-expression closures where the context is clear, use implicit returns:

```swift
attendeeList.sort { a, b in
  a > b
}
```

Chained methods using trailing closures should be clear and easy to read in context. Decisions on spacing, line breaks, and when to use named versus anonymous arguments is left to the discretion of the author. Examples:

```swift
let value = numbers.map { $0 * 2 }.filter { $0 % 3 == 0 }.index(of: 90)

let value = numbers
  .map {$0 * 2}
  .filter {$0 > 50}
  .map {$0 + 10}
```


## Types

+ For primitive Swift types, only specify the object type if it is not set immediately. The object type can be inferred from the value set.
+ `:` should be positioned to the right of the variable name with no spacing.

**Preferred**:
```swift
var title: String
var title = ""

var value: Double
var value = 1.0
```

**Not Preferred**:
```swift
var title: String = ""
var value: Double = 1.0
```

### Constants
Constants are defined using the `let` keyword and variables with the `var` keyword. Always use `let` instead of `var` if the value of the variable will not change.

**Tip:** A good technique is to define everything using `let` and only change it to `var` if the compiler complains!

You can define constants on a type rather than on an instance of that type using type properties. To declare a type property as a constant simply use `static let`. Type properties declared in this way are generally preferred over global constants because they are easier to distinguish from instance properties. Example:

**Preferred**:
```swift
enum Math {
  static let e = 2.718281828459045235360287
  static let root2 = 1.41421356237309504880168872
}

let hypotenuse = side * Math.root2

```
**Note:** The advantage of using a case-less enumeration is that it can't accidentally be instantiated and works as a pure namespace.

**Not Preferred**:
```swift
let e = 2.718281828459045235360287  // pollutes global namespace
let root2 = 1.41421356237309504880168872

let hypotenuse = side * root2 // what is root2?
```

### Static Methods and Variable Type Properties

Static methods and type properties work similarly to global functions and global variables and should be used sparingly. They are useful when functionality is scoped to a particular type or when Objective-C interoperability is required.


### Optionals
Declare variables and function return types as optional with `?` where a `nil` value is acceptable.

Use implicitly unwrapped types declared with `!` only for instance variables that you know will be initialized later before use, such as subviews that will be set up in `viewDidLoad()`. Prefer optional binding to implicitly unwrapped optionals in most other cases.

When accessing an optional value, use optional chaining if the value is only accessed once or if there are many optionals in the chain:

```swift
textContainer?.textLabel?.setNeedsDisplay()
```

Use optional binding when it's more convenient to unwrap once and perform multiple operations:

**Notes:** Swift 5.7 introduced new shorthand syntax for unwrapping optionals into shadowed variables:

**Preferred**:
```swift
if let textContainer {
  // do many things with textContainer
}
```

**Not Preferred**:
```swift
if let textContainer = textContainer {
  // do many things with textContainer
}
```

When naming optional variables and properties, avoid naming them like `optionalString` or `maybeView` since their optional-ness is already in the type declaration.

For optional binding, shadow the original name whenever possible rather than using names like `unwrappedView` or `actualLabel`.

**Preferred**:
```swift
var subview: UIView?
var volume: Double?

// later on...
if let subview, let volume {
  // do something with unwrapped subview and volume
}

// another example
resource.request().onComplete { [weak self] response in
  guard let self else { return }

  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**Not Preferred**:
```swift
var optionalSubview: UIView?
var volume: Double?

if let unwrappedSubview = optionalSubview {
  if let realVolume = volume {
    // do something with unwrappedSubview and realVolume
  }
}

// another example
UIView.animate(withDuration: 2.0) { [weak self] in
  guard let strongSelf = self else { return }

  strongSelf.alpha = 1.0
}
```

* For assigining optional values to a none optional variable with having a default value, use one of our helper methods such as `.orEmpty`, `.orFalse`, `.orZero` and etc.
* For usage of the optional values in a condition, use one of our helper methods: `.isNilOrEmpty`, `.isNilOrFalse`, `.isNilOrZero` and etc.

**Preferred**:
```swift
var flag: Bool?
var totalPrice: Double?

// later on...
if flag.isNilOrFalse {
   price.value = totalPrice.orZero
}

// another example
resource.request().onComplete { [weak self] response in
  guard let self else { return }

  let model = self.updateModel((response?.price?.value).orZero)
  self.updateUI(model)
}
```

**Not Preferred**:
```swift
var flag: Bool?
var totalPrice: Double?

// later on...
if flag != true {
   price.value = totalPrice ?? 0.0
}

// another example
resource.request().onComplete { [weak self] response in
  guard let self else { return }

  let model = self.updateModel(response?.price?.value ?? 0)
  self.updateUI(model)
}
```


### Lazy Initialization
Consider using lazy initialization for finer grained control over object lifetime. This is especially true for `UIViewController` that loads views lazily. You can either use a closure that is immediately called `{ }()` or call a private factory method. Example:

```swift
lazy var locationManager = makeLocationManager()

private func makeLocationManager() -> CLLocationManager {
  let manager = CLLocationManager()
  manager.desiredAccuracy = kCLLocationAccuracyBest
  manager.delegate = self
  manager.requestAlwaysAuthorization()
  return manager
}
```

**Notes:**
  - `[unowned self]` is not required here. A retain cycle is not created.
  - Location manager has a side-effect for popping up UI to ask the user for permission so fine grain control makes sense here.


### Type Inference
Prefer compact code and let the compiler infer the type for constants or variables of single instances. Type inference is also appropriate for small, non-empty arrays and dictionaries. When required, specify the specific type such as `CGFloat` or `Int16`.

**Preferred**:
```swift
let message = "Click the button"
let currentBounds = computeViewBounds()
var names = ["Mic", "Sam", "Christine"]
let maximumWidth: CGFloat = 106.5
```

**Not Preferred**:
```swift
let message: String = "Click the button"
let currentBounds: CGRect = computeViewBounds()
var names = [String]()
```

#### Type Annotation for Empty Arrays and Dictionaries
For empty arrays and dictionaries, use type annotation. (For an array or dictionary assigned to a large, multi-line literal, use type annotation.)

**Preferred**:
```swift
var names: [String] = []
var lookup: [String: Int] = [:]
```

**Not Preferred**:
```swift
var names = [String]()
var lookup = [String: Int]()
```

**NOTE**: Following this guideline means picking descriptive names is even more important than before.


### Syntactic Sugar
Prefer the shortcut versions of type declarations over the full generics syntax.

**Preferred**:
```swift
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```

**Not Preferred**:
```swift
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```


## Functions vs Methods

Free functions, which aren't attached to a class or type, should be used sparingly. When possible, prefer to use a method instead of a free function. This aids in readability and discoverability.

Free functions are most appropriate when they aren't associated with any particular type or instance.

**Preferred**
```swift
let sorted = items.mergeSorted()  // easily discoverable
rocket.launch()  // acts on the model
```

**Not Preferred**
```swift
let sorted = mergeSort(items)  // hard to discover
launch(&rocket)
```

**Free Function Exceptions**
```swift
let tuples = zip(a, b)  // feels natural as a free function (symmetry)
let value = max(x, y, z)  // another free function that feels natural
```


## Memory Management

Code (even non-production, tutorial demo code) should not create reference cycles. Analyze your object graph and prevent strong cycles with `weak` and `unowned` references. Alternatively, use value types (`struct`, `enum`) to prevent cycles altogether.

**Note:** Beside `weak`, by usage of `unowned` we can also prevent strong cycles, however only use `unowned` in cases that is necessary and `weak` can't be used since it's not as safe and could cause unexpected crashes in case garbege collector free up the resource for which it's used.

### Extending object lifetime
Extend object lifetime using the `[weak self]` and `guard let self = self else { return }` idiom. `[weak self]` is preferred to `[unowned self]` where it is not immediately obvious that `self` outlives the closure. Explicitly extending lifetime is preferred to optional chaining.

**Preferred**
```swift
resource.request().onComplete { [weak self] response in
  guard let self else { return }

  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**Not Preferred**
```swift
// might crash if self is released before response returns
resource.request().onComplete { [unowned self] response in
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**Not Preferred**
```swift
// deallocate could happen between updating the model and updating UI
resource.request().onComplete { [weak self] response in
  let model = self?.updateModel(response)
  self?.updateUI(model)
}
```


## Access Control

Full access control annotation in tutorials can distract from the main topic and is not required. Using `private` and `fileprivate` appropriately, however, adds clarity and promotes encapsulation. Prefer `private` to `fileprivate`; use `fileprivate` only when the compiler insists.

Only explicitly use `open`, `public`, and `internal` when you require a full access control specification.

Use access control as the leading property specifier. The only things that should come before access control are the `static` specifier or attributes such as `@IBAction`, `@IBOutlet` and `@discardableResult`.

**Preferred**:
```swift
private let message = "Great Scott!"

class TimeMachine {  
  private dynamic lazy var fluxCapacitor = FluxCapacitor()
}
```

**Not Preferred**:
```swift
fileprivate let message = "Great Scott!"

class TimeMachine {  
  lazy dynamic private var fluxCapacitor = FluxCapacitor()
}
```


## Control Flow

Prefer the `for-in` style of `for` loop over the `while-condition-increment` style.

**Preferred**:
```swift
for _ in 0..<3 {
  print("Hello three times")
}

for (index, person) in attendeeList.enumerated() {
  print("\(person) is at position #\(index)")
}

for index in stride(from: 0, to: items.count, by: 2) {
  print(index)
}

for index in (0...3).reversed() {
  print(index)
}
```

**Not Preferred**:
```swift
var i = 0
while i < 3 {
  print("Hello three times")
  i += 1
}


var i = 0
while i < attendeeList.count {
  let person = attendeeList[i]
  print("\(person) is at position #\(i)")
  i += 1
}
```


### Ternary Operator

The Ternary operator, `?:` , should only be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an `if` statement or refactored into instance variables. In general, the best use of the ternary operator is during assignment of a variable and deciding which value to use.

**Preferred**:

```swift
let value = 5
result = value != 0 ? x : y

let isHorizontal = true
result = isHorizontal ? x : y
```

**Not Preferred**:

```swift
result = a > b ? x = c > d ? c : d : y
```

If the Ternary operator is long and we need to break it into multiple lines, always add an open close paranthesis `()` around the line for increased clearity and better readability.

**Preferred**:

```swift
let myLongValueName = 5
result = (myLongValueName != 0
         ? veryLongVariableNameInThisClass
         : otherVeryLongVariableNameInThisClass)
```

**Not Preferred**:

```swift
let myLongValueName = 5
result = (myLongValueName != 0 ? veryLongVariableNameInThisClass
         : otherVeryLongVariableNameInThisClass)

result = myLongValueName != 0
? veryLongVariableNameInThisClass
: otherVeryLongVariableNameInThisClass
```


## Golden Path

When coding with conditionals, the left-hand margin of the code should be the "golden" or "happy" path. That is, don't nest `if` statements. Multiple return statements are OK. The `guard` statement is built for this.

**Preferred**:
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
  guard let context = context else {
    throw FFTError.noContext
  }
  guard let inputData = inputData else {
    throw FFTError.noInputData
  }

  // use context and input to compute the frequencies
  return frequencies
}
```

**Not Preferred**:
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {
  if let context = context {
    if let inputData = inputData {
      // use context and input to compute the frequencies

      return frequencies
    } else {
      throw FFTError.noInputData
    }
  } else {
    throw FFTError.noContext
  }
}
```

When multiple optionals are unwrapped either with `guard` or `if let`, minimize nesting by using the compound version when possible. In the compound version, place the `guard` on its own line, then indent each condition on its own line. The `else` clause is indented to match the `guard` itself, as shown below. Example:

**Preferred**:
```swift
guard 
  let number1 = number1,
  let number2 = number2,
  let number3 = number3 
else {
  fatalError("impossible")
}
// do something with numbers
```

**Not Preferred**:
```swift
if let number1 = number1 {
  if let number2 = number2 {
    if let number3 = number3 {
      // do something with numbers
    } else {
      fatalError("impossible")
    }
  } else {
    fatalError("impossible")
  }
} else {
  fatalError("impossible")
}
```


## References

* [The Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
* [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/index.html)
* [Using Swift with Cocoa and Objective-C](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/index.html)
* [Swift Standard Library Reference](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/index.html)
