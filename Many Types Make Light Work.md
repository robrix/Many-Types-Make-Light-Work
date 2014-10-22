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

# Approach 1:
# factor implementations _ruthlessly_

^ Every app is broken down into classes, methods, functions, etc. _How_ these break down in a given program is what we refer to as its factoring, just like composite numbers divide into smaller numbers which we call its factors.

^ Following the analogy, there is more than one way to factor a program, and for the purposes of reusing code, some factorings are better than others.

^ What does a good factoring look like?

---

^ The Gang of Four’s _Design Patterns_ describes the design of a hypothetical WYSIWYG document editor called _Lexi_. The discussion is reasonably detailed, addressing the document format, text layout, drawing, scrolling, and cross-platform support.

^ When discussing the latter, they use this wonderful phrase to describe how to support a stable Window interface across multiple platforms with their different ideas of what a window is:

> Encapsulate the concept that varies.
-- Gamma, Helm, Johnson, & Vlissides’ _Design Patterns_

^ Encapsulating—_factoring out_—the thing which we wish to be able to vary is the key here. We may not currently be as concerned about cross-platform support as these authors were, but the principle is the same: factoring out code which we want to change independently is as good a strategy for code reuse as it is for abstraction.

^ As an example of this, let’s consider the data model for a hypothetical aggregating/bookmarking app.

---

# Factor out independent code

```swift
class Post {
	let title: String
	let author: String
	let postedAt: NSDate
	let URL: NSURL
}

class Tweet: Post { … }
class RSS1Post: XMLPost { … }
class RSS2Post: XMLPost { … }
class AtomPost: XMLPost { … }

class XMLPost: Post {
	let XMLData: NSData
	let titlePath: XPath
	let authorPath: XPath
	let postedAtPath: XPath
	let URLPath: XPath
}
```

^ This app’s data model is pretty simple so far. It can pull in posts from Twitter, RSS (in both the 1.x & 2.x branches), and Atom. It doesn’t show the posts’ content per se; just enough to populate a link, or a notification.

^ RSS1, RSS2, & Atom are all XML formats, so they all subclass an `XMLPost` class which uses XPath expressions to parse the necessary information out of the documents. That way the subclasses can just assign or override the properties with the appropriate queries and `XMLPost` will do the rest.

^ This is simple enough, and not an uncommon pattern: a semi-abstract superclass provides a general implementation and its subclasses fill in the blanks where required to support their specific cases.

^ Unfortunately, there’s trouble brewing already. What if the implementation of XPath which we rely on isn’t able to parse large RSS files in a reasonable amount of memory? What if it breaks on feeds which include HTML?

^ It’s not just bugs that can cause this kind of change; features can, too. What if we want to populate post details asynchronously in some cases? What if we want to use a specific parser for a site with a notoriously broken feed, but still represent it as an `RSS2Post` so that the rest of the app works consistently? What if we want to add support for podcasts or appcasts via RSS enclosures? We parse in the XMLPath class, so we’re already using a specific subclass of it by the time we’d have the data together to tell whether there’s an enclosure—egg, meet chicken; chicken, meet egg.

---

# Factoring out independent code

- tightly couples model classes to parsing strategies

- needless margin for error (e.g. initializing abstract classes)

^ There are ways to solve these problems. We could add a subclass of `Post` which wraps an `XMLPost`, adding an enclosure; we could add an `isPodcast` flag to `Post` or `XMLPost`; we could add an optional enclosure property to `Post` or `XMLPost` and have the appropriate views/controllers check for its presence. But these are all working around a problem with the class hierarchy itself: it’s brittle, saying both more and less than what we mean. How so?

^ It says _more_ than what we mean in that it’s too specific. By encoding the parsing strategy in the class hierarchy, we remove our ability to vary it independently of the leaf nodes.

^ It says _less_ than what we mean in that it’s too general. `XMLPost` is abstract, and thus it wouldn’t work for it to be initialized directly; only its subclasses should be. But this isn’t directly expressible in the language, so some tired or unwary developer may make this mistake at some point.

^ Fortunately for us and our somewhat contrived example, a better factoring is pretty simple too.

---

# Factoring out independent code

```swift
class Post {
	let title: String
	let author: String
	let postedAt: NSDate
	let URL: NSURL
}

class Tweet: Post { … }
class RSS1Post: Post { … }
class RSS2Post: Post { … }
class AtomPost: Post {
	init(data: NSData) {
		let parser = XMLParser(data)
		super.init(title: parser.evaluateXPath(…), …)
	}
}

class XMLParser { … }
```

^ Now the various posts all just subclass `Post` directly, while `XMLParser` is completely independent and doesn’t need to know anything about `Post` at all.

^ By factoring parsing out of the `Post` hierarchy, we can vary it independently of the RSS & Atom post classes. We may still have work to do if a replacement for or addition to our parsing strategy doesn’t use the same interface, but by factoring it out in the first place we are free to abstract the variable concept of _parsing_ separately from the orthogonal concept of _data_.

^ Obviously, this is a contrived example; we all know better than to implement XML parsing in our model classes. (Right?) But I know I can think of examples where I’ve conflated the concerns of one class hierarchy with an orthogonal set which _happened_ to coincide—at least at the moment. It’s an easy enough mistake to make, and correcting it proactively like this helps us make our code more flexible to change, bending with it rather than breaking.

^ Even better, doing this setup makes it much easier for us to apply other approaches to reducing brittleness. We didn’t need `XMLPost`; do we need `Post`?

---

# Approach 2:
# share interfaces with protocols

^ In addition to sharing implementations, we often employ subclasses in order to share an interface between distinct implementations—as with `Post` and its subclasses.

^ Superclasses of this nature are often (mostly) abstract. Why do we care whether we use a class interface for this? After all, if a superclass is abstract, then its subclasses aren’t tightly coupled to its implementation details, right?

^ Well, any subclass is coupled to at least _one_ implementation detail of its superclass: which class that even _is_. When a method takes a parameter whose type is of a specific class, it’s almost always overconstraining—tightly coupling. We don’t (and shouldn’t) care that we receive an instance with that specific class’ memory layout and implementation; we care that it has a specific interface. (If we _did_ care about the memory layout, the two classes would _certainly_ be tightly coupled!)

^ This could also needlessly force consumers of our API to jump through hoops when it would be more convenient, more elegant, or more efficient for them to use some other type to implement the interface.

^ Protocols are a way to have our cake and eat it too: we can express the interface we need to interact with _precisely_, and _without_ introducing tight coupling to irrelevant implementation details like what specific class it may be.

---

# Protocols are interfaces

- specify required properties & methods

- Cocoa uses protocols for several purposes

	- delegate/data source protocols (e.g. `UITableViewDelegate`)

	- model protocols (e.g. `NSDraggingInfo`, `NSFetchedResultsSectionInfo`, `NSFilePresenter`)

	- behaviour protocols (e.g. `NSCoding`, `NSCopying`)

^ At the core, protocols are just interfaces, somewhat analogous to a purely abstract class. A protocol specifies required properties & methods. A class which conforms to a protocol must implement all of the required properties & methods in order to compile.

^ Cocoa’s use of protocols can, broadly, be broken down into three categories:

^ First, protocols which delegate some of an object’s behaviour to some other object. `UITableViewDelegate` and `UITableViewDataSource` are this kind of protocol.

^ Second, protocols which resemble a model object, combining a few properties and perhaps some methods around a single theme. This is somewhat more vague than the other two, and not very common in Cocoa; `NSFilePresenter` is an example, combining a presented item’s URL and operation queue with behaviours relating to serialized access to and changes of the item in question.

^ Cocoa also appears to use these in cases where the implementor wants to elide specific type information—we don’t know what particular class is going to be given to us when we receive `NSDraggingInfo` or `NSFetchedResultsSectionInfo`, which means Cocoa avoids vending implementation details via its types, and further avoids compatibility issues when changing the underlying implementations.

^ Third, protocols which describe a single behaviour which an object must be able to perform; for example, conforming to `NSCoding` means that instances of a class can be encoded/decoded; conforming to `NSCopying` means that they can be copied. In Cocoa these typically end in -ing (`NSCopying`, `NSCoding`, `NSLocking`), whereas in Swift’s standard library these typically end in -able (`Equatable`, `Comparable`, `Hashable`).

^ Note that all of these are still just interfaces: they could have used abstract classes instead, but that would constrain the concrete implementations to a specific class hierarchy, which would make using them inconvenient in many cases.

^ So if protocols are shared interfaces, how might we use them to share the key interface in the data model of a contrived aggregator/bookmarking app?

---

# Protocols are shared interfaces

```swift
class Post {
	let title: String
	let author: String
	let postedAt: NSDate
	let URL: NSURL
}

class Tweet: Post { … }
class RSS1Post: Post { … }
class RSS2Post: Post { … }
class AtomPost: Post {
	init(data: NSData) {
		let parser = XMLParser(data)
		super.init(title: parser.evaluateXPath(…), …)
	}
}
```

^ Recall that post-factoring, our app has a flat hierarchy of model classes. Since we aren’t employing implementation sharing (beyond the storage of the properties themselves), this turns out to be _incredibly_ simple to migrate to a protocol instead.

---

# Using protocols to share interfaces

```swift
protocol Post {
	var title: String { get }
	var author: String { get }
	var postedAt: NSDate { get }
	var URL: NSURL { get }
}

class AtomPost: Post {
	let title: String
	let author: String
	let postedAt: NSDate
	let URL: NSURL

	init(data: NSData) {
		let parser = XMLParser(data)
		title = parser.evaluateXPath(…)
		…
	}
}
```

^ I’ve elided `Tweet` & the RSS post classes, but they undergo analogous changes to those made to `AtomPost`. `Post` is changed from a class to a protocol definition; `AtomPost` has constants for its title, &c.; and it binds values to these instead of calling its superclass’ initializer—since it hasn’t got one any more.

^ Note that in Swift, subclassing and conformance to a protocol use the same syntax; this is not a typo! We’re still conveying the same “is a” relationship as we were, but now we’re doing so without coupling `AtomPost` to any particular implementation.

^ `Post` is pretty minimalistic now; it’s a model protocol, but it doesn’t conflate its purpose as a data model with any orthogonal concerns of parsing or presentation. There isn’t any need to factor it further, but what about the other protocols we may have written in our apps: delegate protocols?

---

# Factor your protocols, too

- `UITableViewDelegate` API ref is in _**9 sections**_

- `UITableViewDataSource` & `UITableViewDelegate` aren’t _really_ independent

- exact same problem as with ill-factored classes

- factor around independent concepts instead

^ One complaint with protocols is that it’s easy to end up with long, unwieldy lists of requirements that become a burden to every caller and implementor; every requirement must be implemented by each type implementing the protocol, after all.

^ For example, have you ever written a class implementing every single method in `UITableViewDelegate`?

^ The API ref is broken into 9 sections, but my count it’s more like thirteen different responsibilities including display notifications, selection, editing, and layout—which strays dangerously near to a view responsibility—`UITableViewDataSource` territory.

^ Not only is `UITableViewDelegate` massive, it’s almost inextricably intertwined with `UITableViewDataSource`. Have you ever written a class conforming to `UITableViewDelegate` _or_ `UITableViewDataSource`, but not _both_?

^ Just like with classes, this is a hint that these protocols have too many responsibilities and that they haven’t been divided in the right places. Again just like with classes, we should factor independent concerns out into a separate protocol.

---

# Delegate protocols suggest better factoring

- they force a single class to handle disparate concerns

- instead, “encapsulate the concept that varies”

	- factor out view elements (e.g. rows) instead of delegate methods for contents/behaviour

	- KVO-compliant selected/displayed subset properties (or signals) instead of `will`/`did` delegate callbacks

	- can start by splitting methods into tiny protocols

^ I’d go so far as to say that this applies to nearly every delegate protocol: delegating a kitchen sink of view behaviours to a single object makes it difficult to vary them independently.

^ Instead, we can factor them out: if a view displays multiple rows or cells, make an interface for them instead of using a delegate or a data source.

^ You can put `will`/`did` callbacks for display, selection, etc., in the same interface. If you’ve measured a need for better performance, or if you need to handle e.g. animations at a coarser grain, you can expose signals or KVO-compliant properties for the displayed/selected subset of elements.

^ As a low-effort, medium-reward measure, it might be reasonable to start by splitting e.g. menu or editing interactions off into smaller purpose-specific delegates. If the consumer chooses to implement them all with the same object, they still can; but they’re no longer prevented from using a factoring more appropriate to their application’s design.

^ Note that you can also provide public implementations of these protocols for the default behaviours; this can make it easy for consumers to wrap or otherwise compose them, making the class more convenient to use, more flexible, and simpler to write. It’s easier to understand, as well, since we’ve encapsulated—and documented!—the distinct roles of these interfaces in the API, instead of lumping them in with everything else in the kitchen sink of the delegate protocol.

^ Even better, once we’re using model protocols, we can employ our next approach to help us compose.

---

# Approach 3:
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

# Generic functions over protocols

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

^ Note that this serves the same purpose as a concrete method in an abstract class would. It’s pretty satisfying! One implementation of `second()` is reused for both `Stream` and `List`, and we could do the same for `third()`, `fourth()`, etc., if we needed those; all we needed was a protocol and a generic function.

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

# Approach 4:
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

^ If the set of cases is open-ended, consider using protocol instead.

^ These approaches will help us avoid subclassing when we’re working purely with our own code, but what about when we’re dealing with Cocoa?

---

# Caveat: Cocoa _requires_ you to subclass

^ Many Cocoa classes that we use to form the skeleton of our apps are designed to be subclassed for common uses. `UIViewController` and `UIView` are two of the most common examples, but hardly the only ones. What do we do in these situations?

---

# Write minimal subclasses

^ We may be required to subclass, but that doesn’t mean game over. While we may still be at the whim of new SDKs when it comes to unanticipated change in Apple’s classes, we can defend against the worst.

^ First off, ask yourself if you can configure an instance instead of subclassing. Sometimes classes like `UIViewController` which _typically_ require subclassing can simply be set up instead—whether in a xib, a storyboard, or in code.

- can you configure an instance instead of subclassing?

^ Sometimes the answer is going to be “no.” Likewise, sometimes jumping through hoops to avoid a subclass outright won’t be worth it; in those cases we can apply the same approaches we’ve considered already.

^ First off, we want to make sure that distinct responsibilities are being handled by distinct types. By factoring responsibilities out of the subclass, we avoid making assumptions about the superclass, and no SDK change will ever invalidate an assumption you haven’t made.

- extract distinct responsibilities into their own types

^ You can think of this as coding defensively. In Objective-C, best practice for categories on another party’s types—whether Apple’s or a third-party—is to prefix the method names so as to avoid collisions.

^ Likewise, you can insulate your code from future change—and future coupling!—by minimizing the interfaces that your code operates on, and therefore assumes.

- code defensively

^ It can be quite educational to take this to its logical extreme.

---

# Experiment: access `super` through a protocol

^ For example, you could access your superclass through a protocol: Add a protocol and a property returning self. Now, if you only access your superclass through that property, you have a guaranteed minimum interface—and later on, refactoring the subclass to rely on composition instead of inheritance can be partially accomplished by simply changing the property to store an instance of the superclass instead of returning self.

```swift
protocol DetailViewControllerType {
	var view: UIView { get }
	…
}

extension UIViewController: DetailViewControllerType {}

class DetailViewController: UIViewController {
	var viewController: DetailViewControllerType { self }
	
	// caveat: hooks won’t go through `viewController`
	override func viewDidLoad() {
		// and neither will explicit calls through `super` 
		super.viewDidLoad()
	}
}
```

^ I would be very unlikely to actually ship this code, but I _have_ written it, and learned a lot. When you have some spare time, I recommend trying it out on a branch, ideally with an extant subclass, to see what you can learn about the class and your program as a whole.

^ As with everything we’ve discussed, this is a matter of discipline. It’s the same way with making sure we don’t couple too tightly to our own classes, subclass or otherwise.

---

# A `final` piece of advice

^ To that end, I recommend making all classes `final`—which means “this class cannot be subclassed”—by default.

- make all classes `final` by default

^ Note that I say “by default”—these approaches are tradeoffs, and you may find that this is the wrong one for your specific case. However, by always _defaulting_ to `final`, you ensure that any time you’re removing the keyword, it’s as a conscious decision in light of the circumstances you’re designing for.

- only remove `final` as a conscious choice

^ Further, if you leave a comment as to _why_ the class isn’t `final`, you’ll inform your teammates (and your future self!) of the reasoning behind the decision. Compromise for a deadline is a valid reason, but the reminder can help you to keep in mind that tightly coupling to the superclass should likely still be avoided.

- consider leaving a comment as to why you did

---

# Takeaway

- subclassing is for the weak and timid
- reuse interfaces with protocols
- reuse implementations by factoring & composing

^ The takeaway is pretty simple: subclassing is a weak and brittle substitute for better engineering practices: protocols, minimal types, factoring, and composition.

^ So don’t subclass.

^ I hope you’ve enjoyed the talk! I’d be happy to answer any questions, if we have time.

---

# Thanks to

## Matt Diephouse, Ken Ferry, Kris Markel, Andy Matuschak, Ryan McCuaig, Kelly Rix, Justin Spahr-Summers, Patrick Thomson…

## …and you ❤️

![](http://upload.wikimedia.org/wikipedia/commons/d/db/Railroad_Coupling_(CMRR\).jpg)
