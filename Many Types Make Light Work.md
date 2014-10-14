# Many Types Make Light Work

### @rob_rix ❦ rob.rix@github.com

---

# We want to minimize effort;
# we have to reuse code.

^ One of the most important parts of programming is recognizing patterns.

^ We don’t want to write the same—or nearly the same—code multiple times for each case we’re using. We want to reuse code.

^ This results in simpler, easier-to-understand systems with fewer, easier-to-find bugs.

^ There are two basic approaches that we use for this in Objective-C & Swift:

---

# 1. Share: subclass

^ We can reuse code by subclassing. If our class does a thing and we need almost the same thing elsewhere, we can subclass the original class, or extract a common superclass and then subclass that.

# TODO: subclass diagram

^ For example, if we have a large view controller that we want to reuse part of, we could extract the common bit into a superclass. Now the original class can subclass that, and the new one can too.

---

# 2. Separate: compose

^ We can also separate the common bits into one class, and the distinct bits into other ones which we compose with the common one.

# TODO: compose diagram

^ These diagrams are quite similar. One shows relationships between classes, the other shows relationships between objects, but they’re both solving the same problem.

^ However:

---

# Change is inevitable

^ Requirements change. We have an idea for a feature. We get a bug report. We get new design mockups. Apple releases a new version of the operating system and the SDK breaks some of our assumptions. Apple releases new hardware or APIs that we want to use. Apple releases a new _language_!

^ Some change will always be required; necessary change is inevitable.

---

# We want to minimize effort;
# we need to minimize the impact of necessary change.

^ Change is inevitable, but it’s not always our choice: Apple’s OS updates may force our hand.

^ Either way, we want to be able to act on it efficiently.

---

# Coupling is when change _here_ forces change _there_.

![right](http://upload.wikimedia.org/wikipedia/commons/f/f1/Train_coupling.jpg)

^ Coupling is a problem. These linked railway couplings ensure that if one car moves—changes its position—the other moves with it.

---

# We want to minimize effort;
# we have to reduce coupling.

![right](http://upload.wikimedia.org/wikipedia/commons/f/f1/Train_coupling.jpg)

^ This is an apt metaphor for our code. To as great an extent as possible, changing one part of our program should not require us to change other parts of our program. If the code is tightly coupled, changes cascade.

^ However:

---

# Subclassing encourages tight coupling.

^ Our diagrams had the same shape but the arrows hide the fact that the subclasses always depend on implementation details of their superclasses.

^ This is almost universal. Changing a class’ superclass is going to break it sooner or later. Each and every subclass increases the potential fallout.

^ Suddenly our strategy for code reuse becomes a liability.

^ Fortunately there’s another, simpler strategy which will help us deal with this problem:

---

# Don’t subclass.

^ The reason that subclassing couples tightly is that it conflates reusing an interface with reusing an implementation.

^ This is unavoidable; it’s what subclassing _is_. You get all of the superclass’ behaviour, whether you want it or not, unless you have specifically overridden it and taken care to do so in a manner compatible with its design. And that’s still only valid until the _next_ time it changes.

^ We’re going to look at some approaches to writing more flexible, reliable, & maintainable code by not subclassing; we’ll look at them individually, but they aren’t mutually exclusive.

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

^ A common complaint with protocols is that you either end up with long, unwieldy lists of requirements that become a burden to anything implementing them. Every required method you add has to be implemented by each implementor, every optional method has to be checked for at runtime.

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

# Approach 3.:
# functions

^ Swift’s function overloading, generic functions, and function types can all be used as shared interfaces.

---

# Function overloading as an (insufficient) interface

^ Swift supports multiple dispatch: which function will be executed when you call a function can depend on both the argument and return types. That means that ordinary, i.e. _free_ functions can act a lot like methods: you can write functions `first(…)` and `dropFirst(…)` taking `Stream` and another pair by the same names taking `List`.

```swift
// In the Swift standard library
func first<C : CollectionType>(x: C) -> C.Generator.Element?
func dropFirst<Seq : Sliceable>(s: Seq) -> Seq.SubSlice

// In our code
func first(stream: Stream<T>) -> T? { … }
func dropFirst(stream: Stream<T>) -> Stream<T>

func first(list: List<T>) -> T? { … }
func dropFirst(list: List<T>) -> List<T>
```

^ Now we can call `first(…)` and pass in a value of a type conforming to `CollectionType` (e.g. an `Array`), or a `Stream`, or a `List`, and we’ll have the same behaviour each time.

^ `first()` and `dropFirst()` are all we need to write functions like `second()`, `third()`, etc. without introducing new primitive operations. But while we can see that `Stream`, `List`, and `Sliceable` (which conforms to `CollectionType`) can all be passed to `first()` and `second()`, the compiler hasn’t gotten the hint that these share an interface, so we would have to implement `second()` once for `List`, once for `Stream`, and once for `Sliceable`.

# But we can’t write `second(…)` generically!

^ Since function overloading isn’t enough to write `second()` generically, we need to use a `protocol`.

---

# Generic functions over `protocol`s

^ By introducing a `protocol`, `ListType`, capturing the `first()` and `dropFirst()` requirements, we can implement `second()`, etc, as free functions, generic over `ListType`:

```swift
protocol ListType {
	typealias Element
	func first() -> Element?
	func dropFirst() -> Self
}

func second<L: ListType>(list: L) -> Element? {
	return list.dropFirst().first()
}
```

^ This serves the same purpose as a concrete method in an abstract class would. This protocol doesn’t encompass `Sliceable`, so we’d still have to write an implementation for it if we wanted one, but by conforming to `ListType, both `List` and `Stream` can share this single implementation of `second()`.

^ This is satisfying: there’s less code to write & maintain, and we less code to write if we want to extend it to `third()` through `twentySixth()`. Sometimes, though, you can reduce it even further; sometimes all you need is a single function.

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

^ These approaches are a big help in writing your own code, but often it’s not your class that you need to subclass.

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
- Justin Spahr-Summers
- you ❤️

![](http://upload.wikimedia.org/wikipedia/commons/d/db/Railroad_Coupling_(CMRR\).jpg)
