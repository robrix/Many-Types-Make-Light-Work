# Many Types Make Light Work

### @rob_rix ❦ rob.rix@github.com

^Every programmer regularly faces the challenges of managing complexity and change. Subclassing can help us simplify, by sharing code and interfaces between related parts of our program. But when requirements change out from under our class hierarchy, unanticipated fragility can mean bugs, sweat and tears. Swift offers us a better way.

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

# Approach 2.5:
# factor `protocol`s _ruthlessly_

^ A common complaint with protocols is that you either end up with long, unwieldy lists of requirements that become a burden to anything implementing them. Every required method you add has to be implemented by each implementor, every optional method has to be checked for at runtime.

^ These problems are a hint that your `protocol`s have too many responsibilities. Break them down just like your implementations—factor out every independent concern into a separate `protocol`.

^ When you type a parameter or a property with more than one protocol, consider whether that’s really what you mean: generally two protocol requirements is a sign that you want two parameters or properties.

^ (The exception is e.g. a collection of elements which you need to be both `Equatable` and `Printable`; these kinds of …`able` protocols are usually stating something fundamental about the type more so than just declaring an interface for communication with it.)

---

# Approach 3.:
# function types

^ Swift supports multiple dispatch: which function will be executed when you call a function can depend on both the argument and return types. That means that ordinary, i.e. _free_ functions can act a lot like methods: you can write a function `foo(…)` taking `Bar` and another taking `Quux`.

---

# Function overloading shares code for `protocol`s

^ This is also the preferred way of implementing behaviour common to protocols. For example, a `StreamType` protocol with `first` and `dropFirst` method requirements would probably not want to also require `second`; instead, a free function `second` could be implemented generically, parameterized by a type conforming to `StreamType`.

```swift
protocol StreamType {
	typealias Element
	func first() -> Element?
	func dropFirst() -> Self
}

func second<S: StreamType>(stream: S) -> Element? {
	return stream.dropFirst().first()
}
```

---

# Function types are shared interfaces

^ Simple enough interfaces can be expressed with just a function type or two. For example, Swift’s built-in `GeneratorOf` is a `GeneratorType` which you can construct using either an existing `GeneratorType`, or a function:

```swift
struct GeneratorOf<T> : GeneratorType {
	init(_ nextElement: () -> T?)

	init<G : GeneratorType where T == T>(_ base: G)
	…
}
```

^ When you construct a `GeneratorOf` wrapping another generator, it wraps it up in a closure which it passes to the other initializer; that closure is the necessary and sufficient interface shared by anything it wraps.

---

# Approach 4.:
# (many) minimal types

^ Similarly, it’s reasonable to ask yourself just how much you need to abstract. Does your `Author` type really need a `Bibliography` instead of a list of publications?

^ Minimal types are ones where you can’t really factor them out any further.

---

# `Stream` as a protocol

^ Earlier we looked at a protocol, `StreamType`.

```swift
protocol StreamType {
	typealias Element
	func first() -> Element?
	func dropFirst() -> Self
}
```

# Do we need more than one `StreamType`?

^ But do we really need more than one kind of `Stream`?

---

# `Stream` as a minimal type

^ There isn’t a lot of useful axis for variance here. A direct implementation is pretty trivial:

```swift
enum Stream<T> {
	case Cons(Box<T>, () -> Stream)
	case Nil
}

func first<T>(stream: Stream<T>) -> T? {
	…
}

func dropFirst<T>(stream: Stream<T>) -> Stream<T> {
	…
}
```

^ We can also implement `first()` and `dropFirst()` as free functions, now, to go with the Swift standard library’s implementations over `CollectionType` and `Sliceable`.

^ With this in our toolbox, other types can compose with or return `Stream<T>` as-is. Another way of describing minimal types, then, might be that minimal types are ones which you’d never need to subclass.

^ On the other hand, sometimes we need _limited_ variance. `Stream` here is an example of that in the sense that there are two cases to think about when using streams: empty and non-empty. We use `enum` to cover those here, and that is more generally applicable as well.

---

# `enum`s are shared interfaces

^ An `enum` provides a fixed set of cases, which provide the alternative values for the type.

Best for fixed sets of alternatives:

```swift
enum Natural {
	case Zero
	case Successor(Box<Natural>)

	var toInt: Int {
		switch self {
		case Zero:
			return 0
		case let Successor(x):
			return 1 + x.value.toInt
		}
	}
}
```

^ This is a particularly good use case for `enum` since there are only two kinds of natural numbers possible; we don’t need to worry about adding more kinds later on and having to change any code using them to match.

^ If the set of cases is open-ended, consider using `protocol` instead.

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
- Ryan McCuaig
- you ❤️

![](http://upload.wikimedia.org/wikipedia/commons/d/db/Railroad_Coupling_(CMRR\).jpg)
