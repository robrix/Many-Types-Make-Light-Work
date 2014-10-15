# Many Types Make Light Work

### @rob_rix ❦ rob.rix@github.com

^ I’ll be talking about code reuse and subclassing in Swift today. We’ll take a quick look at the whys and hows of reusing code by subclassing; at some problems with subclassing; what we can do instead of subclassing; and what we can do to mitigate the problems with subclassing.

---

# Managing complexity under change

^ One of the most important parts of programming is recognizing patterns. Patterns are opportunities for us to reuse code, reducing the complexity of our system.

^ This is inherently in tension with another major part of programming: adding features. Programming is a kind of balancing act of managing this complexity under change. The key way for us to manage the complexity of a program is to reuse code.

^ In Swift & Objective-C, reusing _code_ typically means either reusing _implementations_ or reusing _interfaces_. We’ll take a quick look at each of these.

---

# Reusing implementations

^ When we’ve written a method in one class and we need that same behaviour in another, ideally we wouldn’t copy and paste the method into the new context. Instead, we’d extract the common code into a function or method somewhere we can call it from both of the points which need its behaviour. That’s what reusing implementations means.

- Don’t Repeat Yourself (DRY)

^ We call this “Don’t Repeat Yourself” or “DRY,” and at the core, it’s not much more complicated than that: we abstract bits of our programs, and package them up for reuse elsewhere.

- functions, methods, classes, categories, etc. let us reuse implementations

- subclasses inherit their superclass’ implementation

---

# Reusing interfaces

^ We also want to be able to handle similar objects with the same code. A method taking a string doesn’t need to know if it’s mutable or not to ask it for its length; instead, `NSString`, `NSMutableString`, and any subclasses thereof all share an interface.

- can call methods without knowing an instance’s class

^ A class declares the interface through which we can manipulate its instances.

- subclasses inherit their superclass’ interface

^ We can also describe interfaces in the abstract, without anchoring them to a specific implementation using protocols. Protocols provide an interface, and a concrete implementation is provided by a class.

- protocols describe an interface without tying it to an implementation

^ Now that we’re on the same page about interface & implementation reuse, we’ll look at how we do them. Broadly speaking, the most common ways that we reuse interfaces & implementations are subclassing, and composition.

---

# Subclassing

^ Subclassing is immediately familiar: a subclass inherits its superclass’ interface & implementation.

- subclasses inherit their superclass’ interface

^ This makes subclassing a convenient way to group similar things together under a common (superclass) interface, thus reusing it.

- subclasses inherit their superclass’ implementation

^ It also makes it convenient to reuse the implementations provided by the superclass—any functionality we put into the base class will be shared by any subclass which doesn’t explicitly override it.

- “is a”

^ Subclassing defines an “is a” relationship—we say that `UITableView` is a `UIView` because it subclasses `UIView`. This is the key contrast with composition, which we’ll look at next.

---

# Composition

^ Composition is a less familiar term, but you use the concept all the time: it basically means using objects together. For example, a `UIView` is composed with its superview, its subviews, and its layer; a `UIViewController` is composed with its parent controller, its child controllers, and its views; and so on.

- composition is ~~when you put a thing in a thing and then it’s a thing then~~ using objects together

^ In contrast with the “is a” relationship defined by subclassing, this style of composition defines a “has a”/“has many” relationship, e.g. a `UITableView` has a `UITableViewDelegate`, and it has many `UITableCellView`s.

- properties; “has a/many”

^ We can also consider a class as being composed with the types it interacts with but doesn’t own, i.e. those which it receives as parameters, or creates and gives to something else.

^ This defines a “uses” sort of relationship more so than a “has” one, e.g. `UIApplication` uses `NSNotificationCenter`, but ultimately it’s just another example of composition.


- parameters; “uses”

---

# The trouble with subclassing

^ The perceived convenience of subclassing comes at a cost: if we want to reuse the interface, the implementation tags along anyway.

- it _conflates_ reusing interfaces with reusing implementations

^ This means that every change to the superclass affects each subclass. If a change invalidates some assumption of a subclass, that subclass now has a bug from a change in another piece of code.

^ If we’re lucky, the type system catches it and we can’t compile until we fix it. If we’re unlucky, it slips past our unit tests and QA and App Store review, and our customers encounter it.

^ For example, on OS X Mavericks, `NSViewController` doesn’t have the `-viewWillAppear`, `-viewDidAppear`, etc. methods which we’re familiar with from `UIViewController`. A subclass could, however, implement those methods and call them at the appropriate times. But on Yosemite, we have a bug: these methods are called twice: once by our code, and once by our superclass.

- it _couples_ subclasses to superclass implementations

^ It also encourages other code to make more assumptions about subclasses than would otherwise be possible, simply because the interfaces are overly broad—and broader with each subclass. This can lead to even more coupling and brittleness, further increasing the risk and cost of change.

- it _encourages_ tight coupling in composed classes

![right](http://upload.wikimedia.org/wikipedia/commons/f/f1/Train_coupling.jpg)

^ It’s important to note that subclassing is solving the same problems as composition does—it is a way of reusing interfaces and implementations. That gives us a pretty simple solution to these problems.

---

# Don’t subclass.

^ In the abstract, subclassing is _unnecessary_. Composition lets us have our cake and eat it too: we can reuse interfaces and implementations at our discretion without automatically coupling tightly.

^ It’s still possible for us to write tightly coupled code _without_ subclassing, of course, but it’s easier for us to decouple code in the absence of subclassing than in its presence.

^ To that end, we’re going to look at some approaches to writing  more flexible, reliable, & maintainable code by not subclassing. While these are presented separately, they aren’t mutually exclusive; you can mix and match to fit the task at hand.

---

# Approach 1.:
# factor implementations _ruthlessly_

^ Chances are your classes are too big. Follow the One Responsibility Rule: break them down into one class per responsibility and use those together instead.

^ Factoring helps us reduce subclassing most when we’re subclassing in order to share an implementation. For example, `UIView` and `NSView` implement common behaviours which many views will need, e.g. drawing, converting geometry between coordinate systems, animation, appearance lifecycle, event handling; a better factoring of these would enable us to use precisely the behaviours we want without subclassing & potentially gaining undesirable behaviours as well.

^ Further, we’d be that much better insulated by default from unanticipated change in Apple’s frameworks.

^ This sort of subclassing is common within our own codebases as well, e.g. model classes often form hierarchies of this nature. Split them up & factor them out; the resulting flexibility often ends up reducing the size of the codebase (and thus surface area for bugs) as well.

^ Implementation sharing is not the only reason we subclass, though, so other approaches are needed too.

---

# Approach 2.:
# share interfaces with `protocol`s

^ We also employ subclasses in order to share an interface between distinct implementations. Superclasses of this nature are often (mostly) abstract. Foundation’s `NSValueTransformer` is an example of this, and we can look at Core Data’s `NSManagedObject` this way as well.

^ When we take a parameter whose type is a specific class, we’re  almost always overconstraining—tightly coupling. We don’t (and shouldn’t) care that we receive an instance with that specific class’ memory layout and implementation; we care that it conforms to a specific interface.

^ Say what you mean _precisely_ by expressing those interfaces as `protocol`s.

---

# …but factor your `protocol`s ruthlessly, too

^ A common complaint with protocols is that you end up with long, unwieldy lists of requirements that become a burden to anything implementing them. Every required method you add has to be implemented by each implementor, every optional method has to be checked for at runtime.

^ For example, have you ever written a class implementing every single required method in `UITableViewDelegate`?

- `UITableViewDelegate` API ref is in _**9 sections**_

^ The API ref is broken into 9 sections, but my count it’s more like thirteen different responsibilities including display notifications, selection, editing, and layout—which strays dangerously near to a view responsibility—`UITableViewDataSource` territory.

^ Not only is `UITableViewDelegate` massive, it’s almost inextricably intertwined with `UITableViewDataSource`. Have you ever written a class conforming to `UITableViewDelegate` _or_ `UITableViewDataSource`, but not _both_?

- `UITableViewDataSource` & `UITableViewDelegate` aren’t _really_ independent

^ Just like with classes, this is a hint that these `protocol`s have too many responsibilities and that they haven’t been divided in the right places. Again just like with classes, we should factor out every independent concern into a separate `protocol`.

^ That wouldn’t necessarily mean thirteen (or more!) `protocol`s for `UITableView`, either—the display notifications and height calculations are split between cells, headers, and footers, but they don’t have to be. Likewise, the data source is more or less a `Stream` of (hypothetical) `UITableSection`s.

- Swift does better: `Comparable`, `Equatable`, `GeneratorType`, etc. each have **1** requirement

^ The takeaway is that the same forces which lead to MVC meaning Massive View Controller will affect your `protocol`s too, if you let them. Fortunately, the One Responsibility Rule is a good rule of thumb here, too.

- “kitchen sink” `protocol`s are avoidable; factor

---

# `protocol`s are open-ended shared interfaces

Use `protocol` for open-ended interfaces:

```swift
protocol VehicleType {
	var capacity: Int
	var passengers: [Person] { get }
}

class Train: VehicleType { … }
class Plane: VehicleType { … }
class Automobile: VehicleType { … }
```

---

# Approach 3.:
# functions

^ We can also use Swift’s functions to reuse code. Next, we’ll look at function overloading, generic functions, and function types; let’s start with overloading.

---

# Function overloading is almost an interface

^ Swift supports multiple dispatch: which function will be executed when you call a function can depend on both the argument and return types.

^ That means that ordinary, i.e. _free_ functions can act a lot like methods: you can write functions `first(…)` and `dropFirst(…)` taking `Stream` and another pair by the same names taking `List`.

- `first(…)` returns the first element of a stream/list

```swift
func first<T>(stream: Stream<T>) -> T? { … }
func first<T>(list: List<T>) -> T? { … }
```

- `dropFirst(…)` returns the rest of a stream/list following the first element

```swift
func dropFirst<T>(stream: Stream<T>) -> Stream<T> { … }
func dropFirst<T>(list: List<T>) -> List<T> { … }
```

^ Now we can call `first()` and pass in either a `Stream` or a `List` and we’ll get the behaviour we want.

---

# Function overloading is not really an interface

^ If we want to write a function, `second()`, which takes a `List` or a `Stream` and returns the second element, then all we need is to call `first()` on the result of calling `dropFirst()`—we don’t need any new primitive operations. Unfortunately, if we try to write that function, we run into a problem: what is the type of its parameter?

- `second(…)` returns the second item in a list or stream

^ _We_ can see that `Stream` and `List` can be passed to `first()` and `dropFirst()`. However, `first()` and `dropFirst()` aren’t part of an _interface_; we’re just using them ad hoc. If we wanted to write `second()` for lists and streams as-is, we’d have to implement it twice—once for `List`, and again for  `Stream`.

- But we can’t write `second(…)` generically without a real interface

```swift
func second<T>(…?!) -> T? { … }
```

^ What we need here is the combination of a protocol—that is, an interface—and a generic function. Fortunately, Swift gives us both of these.

---

# Generic functions over `protocol`s

^ Here we have a protocol named `ListType`, which captures `first()` and `dropFirst()` as requirements.

```swift
protocol ListType {
	typealias Element
	func first() -> Element?
	func dropFirst() -> Self
}
```

^ This gives us a named interface which we can use to implement `second()` generically as a free function over values of `ListType`:

```swift
func second<L: ListType>(list: L) -> Element? {
	return list.dropFirst().first()
}
```

^ This serves the same purpose as a concrete method in an abstract class would. This protocol doesn’t encompass `Sliceable`, so we’d still have to write an implementation for it if we wanted one, but by conforming to `ListType, both `List` and `Stream` can share this single implementation of `second()`.

^ This is satisfying: one implementation of `second()` is reused for both `Stream` and `List`, and we could do the same for `third()`, `fourth()`, etc., if we needed those; all we needed was a protocol and a generic function.

^ That’s pretty cool, but in some cases, all you need is the function.

---

# Function types are shared interfaces

^ Every function type is a minimal interface, taking something (possibly `Void`) and returning something (possibly `Void`). That means that simple enough interfaces can be expressed with just a function type or two. For example, Swift’s built-in `GeneratorOf` is a `GeneratorType`—an iterator—which you can make wrapping some other generator, or wrapping a function:

```swift
struct GeneratorOf<T> : GeneratorType {
	init(_ nextElement: () -> T?)

	// A convenience to wrap another GeneratorType
	init<G : GeneratorType where T == T>(_ base: G)
	…
}
```

^ `GeneratorOf` doesn’t mention the `GeneratorType` it wraps anywhere but in the type parameters of the convenience init. That tells us that it doesn’t store it in a property (since that property would need to have that type).

^ Instead, when you make a `GeneratorOf` wrapping another generator, it wraps it up in a closure which it passes to its other initializer; that closure has the necessary and sufficient interface shared by anything it wraps: it takes `Void` and returns an optional element.

^ If you can wrap up any function from `Void` to `Optional<T>` in a `GeneratorOf` with no other feedback provided, why does `GeneratorType` exist at all? Why don’t they _only_ use function types?

^ Alas, that one will have to wait for the Swift team to write their memoirs. We can assume that they have their reasons, though, and so might we—sometimes you need just a _little_ extra functionality, even once you’ve factored your types out as far as you can.

---

# Approach 4.:
# (many) minimal types

^ Similarly, it’s reasonable to ask yourself just how much you need to abstract. Does your `Author` type really need a `Bibliography` instead of a list of publications?

^ Minimal types are ones where you can’t really factor them out any further.

---

# `List` as a protocol

^ For example, we could describe lists using a protocol:

```swift
protocol ListType {
	typealias Element
	init(first: Element, rest: Self?)
	var first: Element { get }
	var rest: Self?
}
```

^ But doing so raises a question.

---

# `List` as a minimal type

^ Lists are about as simple as it gets. A direct implementation is pretty trivial:

```swift
enum List<T> {
	case Cons(Box<T>, Box<List<T>>)
	case Nil(Box<T>)

	var first: T { … }
	var rest: List<T>? { … }
}
```

_fin_

^ There isn’t a lot of useful variance possible here. We could have used an `struct` or a `class`; we could have made the `Nil` case not have a value and made `first` optional; but these are implementation details, and irrelevant to our API’s consumers.

^ Aggressive factoring can remove some of the need for sharing interfaces between multiple types. With `ListType`, consumers of the API are concerned with whether something _is_ a list, whereas with `List<T>` _we_ are concerned with whether it _has_ a list.

^ Further, by conforming `List<T>` to `SequenceType`, we can avoid any need for consumers to care that we have a list at all—all they need to know is that they have a way to access its elements sequentially; that makes our choice of `List<T>` (vs. `Array<T>`) itself an implementation detail.

^ On the other hand, sometimes we need some small, fixed amount of variance. This can be a great use case for `enum`.

---

# `enum`s are fixed shared interfaces

^ An `enum` provides a fixed set of cases, which provide the alternative values for the type.

^ For example, here’s a `Result` type for operations which can end either in a successful value or an error:

Use `enum` for fixed sets of alternatives:

```swift
enum Result<T> {
	case Success(Box<T>)
	case Failure(NSError)
}
```

^ This is a particularly good use case for `enum` since there are only two possibilities: it either worked or it didn’t. We don’t need to worry about adding more cases later on and having to update every function using `Result` to match.

^ If the set of cases is open-ended, consider using `protocol` instead.

---

# Problem: Cocoa _requires_ you to subclass

^ `UIView`, `UIViewController`, `NSManagedObject`, `NSTableView`, and other classes were designed to be subclassed. Sometimes you can get away with using an existing subclass like `UINavigationController` or a delegate protocol like `NSTableViewDelegate`, but often the only route to customization is to subclass.

^ We still want to minimize effort & share code; how do we do that?

---

# Solution: Write minimal subclasses

^ We can apply several of the same approaches.

---

# Factor _ruthlessly_!

^ Any code you might want to share belongs somewhere other than in this subclass. Leave as little code as possible in the subclass.

---

# Mark subclasses as `final`

^ Don’t succumb to the temptation to subclass your subclass.

---

# Thanks

- Matt Diephouse
- Kris Markel
- Andy Matuschak
- Ryan McCuaig
- Kelly Rix
- Justin Spahr-Summers
- Patrick Thomson
- you ❤️

![](http://upload.wikimedia.org/wikipedia/commons/d/db/Railroad_Coupling_(CMRR\).jpg)
