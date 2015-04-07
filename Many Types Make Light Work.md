# Many Types Make Light Work[^1]

### 🐦 [@rob_rix](https://twitter.com/rob_rix)<br>🐙 [@robrix](https://github.com/robrix)<br>📩 [rob.rix@github.com](mailto:rob.rix@github.com)

^ who am I

^ where do I work

^ what am I going to talk about

[^1]: https://github.com/robrix/Many-Types-Make-Light-Work

---

# Reusing code

- change happens: bugs, features, platform changes

- complexity scales with quantity of code

- programming is managing complexity under change

- reusing code is necessary (if insufficient)

- by “reusing _code_,” we mean…

	- reusing _implementations_ 

	- reusing _interfaces_

^ it’s pretty rare to be finished an app: fixing bugs, adding features, keeping pace with platform changes

^ adding code tends to increase the complexity & therefore risk—more bugs to fix, more change, turtles all the way down

^ one necessary ingredient in managing complexity is reusing code—code we can reuse doesn’t increase the complexity further

^ “reusing code” is too vague; specifically we reuse _implementations_ & reuse _interfaces_

---

# Reusing implementations

- Don’t Repeat Yourself (DRY)

- functions, methods, (sub)classes, &c.

# Reusing interfaces

- use the same code with different types

- subclassing, protocols

^ Reusing implementations boils down to “Don’t Repeat Yourself” or “DRY.” We factor out code we want to reuse into a function, method, or class, and call it from the places which need its behaviour. One function, used by multiple callers.

^ Reusing interfaces is the flip side of the coin. A function which takes a parameter of a given class can take any subclass, because a class defines an interface shared—reused—by its subclasses. Likewise, a function which takes a parameter of a given protocol can take any concrete type conforming to it—that’s what protocols are for. One function, taking multiple types.

^ Broadly speaking, the most common tools we employ for code reuse are subclassing & composition.

---

# Subclassing

- inherit superclass’ interface & implementation

- describes the class hierarchy at compile time

# Composition

- composition is ~~when you put a thing in a thing and then it’s a thing then~~ using objects together

- describes the object graph at runtime

^ Subclassing is immediately familiar: a subclass inherits its superclass’ interface & implementation. That makes subclassing convenient both for grouping similar things together under a shared interface, and for sharing the superclass’ functionality between multiple subclasses.

^ Composition may be a less familiar term, but you use the concept all the time: it boils down to objects which can be used together. For example, a `UIView` is composed with its superview, its subviews, and its layer; a `UIViewController` is composed with its parent controller, its child controllers, and its views; and so on.

^ We reuse implementations using composition by simply using the different implementations together. Composition doesn’t provide interface reuse—we’d describe that as the job of its fraternal twin, abstraction—but it does work just fine with shared interfaces: anything which composes with a given interface can compose with any type providing it.

^ But even though we _can_ use composition & abstraction to reuse implementations & interfaces, we will often reach for subclassing first. However, they’re not quite equivalent…

---


> subclassing ≃ 💥🔥💀

^ …and unfortunately, subclassing tends to cause us problems. For example…

---

> Subclassing _conflates_ reuse of interfaces with reuse of implementations

^ The perceived convenience of subclassing comes at a cost: if we want to reuse the interface, or just part of the implementation, the rest of the implementation tags along anyway.

---

> Subclassing _couples_ superclass and subclass implementations

^ This means that every change to the superclass affects each subclass. If a change invalidates some assumption of a subclass, that subclass now has a bug from a change in another piece of code. Likewise, if the superclass calls its own methods (as they tend to), the subclass can also invalidate an assumption of the superclass—even if that assumption is new.

^ For example, on OS X Mavericks, `NSViewController` doesn’t have the `-viewWillAppear`, `-viewDidAppear`, etc. methods which we’re familiar with from `UIViewController`. A subclass could, however, implement those methods and call them at the appropriate times. But under Yosemite, `NSViewController` adds and calls those methods, meaning we now have a bug: these methods are called twice: once by our code, and once by our superclass. All we did is compile against the new SDK.

---

> Subclassing _encourages_ tight coupling in composed classes

^ Subclassing also enables other code using the hierarchy to make more assumptions about subclasses than would otherwise be possible, simply because the interfaces are broader than they need to be—and they get broader with each layer of subclass. This can lead to even more coupling and brittleness, unintentionally increasing the risk and cost of change (whether on our part or Apple’s).

^ The good news is that ultimately, subclassing and composition/abstraction solve the same problems—reusing interfaces, and reusing implementations. That gives us a pretty simple solution to this kind of problem.

---

> Don’t subclass.
— me, here, just now

^ In the abstract, subclassing is _unnecessary_. Composition lets us have our cake and eat it too: we can reuse interfaces and implementations at our discretion without automatically coupling tightly.

^ It’s still possible for us to write tightly coupled code _without_ subclassing, of course, but it’s easier for us to decouple code in the absence of subclassing than in its presence.

^ Of course, easier said than done. We’ve been trained to subclass by our peers, mentors, books, blog posts, code bases, and by the frameworks and languages themselves. But it doesn’t have to be that way, and Swift makes not subclassing easier than ever.

^ To that end, we’re going to look at some approaches to writing  more flexible, reliable, & maintainable code by not subclassing. While these are presented separately, they aren’t mutually exclusive; you can mix and match to fit the task at hand.

---

# Approach 1:
# Factor class hierarchies out

^ Every app is broken down into classes, methods, functions, etc. _How_ these break down in a given program is what we refer to as its factoring, just like composite numbers divide into smaller numbers which we call its factors.

^ Following the analogy, there is more than one way to factor a program, and for the purposes of reusing code, some factorings are better than others.

^ What does a good factoring look like?

---

^ The Gang of Four’s _Design Patterns_ describes the design of a hypothetical WYSIWYG document editor called _Lexi_. The discussion is reasonably detailed, addressing the document format, text layout, drawing, scrolling, and cross-platform support.

^ When discussing the latter, they use this wonderful phrase to describe how to support a stable Window interface across multiple platforms with their different ideas of what a window is:

> Encapsulate the concept that varies.
— Gamma, Helm, Johnson, & Vlissides’ _Design Patterns_

^ Encapsulating—_factoring out_—the thing which we wish to be able to vary is the key here. We may not currently be as concerned about cross-platform support as these authors were, but the principle is the same: factoring out code which we want to change independently is as good a strategy for code reuse as it is for abstraction.

^ As an example of this, let’s consider the data model for a hypothetical aggregating/bookmarking app. It already manages tweets, and now we want to add RSS to it as well.

---

# Factor out independent code

```swift
// Existing model classes
class Post {
	let title: String
	let author: String
	let postedAt: NSDate
	let URL: NSURL
}

class Tweet: Post { … }

// New RSS support
class XMLPost: Post {
	let XMLData: NSData
	let titlePath: XPath
	…
}

class RSS1Post: XMLPost { … }
class RSS2Post: XMLPost { … }
```

^ This app’s data model is pretty simple to begin with. It can pull in posts from Twitter, and now we’re adding RSS support.

^ RSS1 & RSS2 are XML formats, so they both subclass an `XMLPost` class which uses XPath expressions to parse the necessary information out of the documents. That way the subclasses can just assign or override the properties with the appropriate queries and `XMLPost` will do the rest.

^ This is straightforward enough, and not an uncommon pattern: a semi-abstract superclass provides a general implementation and its subclasses fill in the blanks where required to support their specific cases.

^ Unfortunately, there’s trouble brewing already. What if the implementation of XPath which we rely on isn’t able to parse large RSS files in a reasonable amount of memory? What if it breaks on feeds which include HTML?

^ It’s not just bugs that can cause this kind of change; features can, too. What if we want to populate post details asynchronously in some cases? What if we want to use a specific parser for a site with a notoriously broken feed, but still represent it as an `RSS2Post` so that the rest of the app works consistently? What if we want to add support for podcasts or appcasts via RSS enclosures? We parse in the XMLPath class, so we’re already using a specific subclass of it by the time we’d have the data together to tell whether there’s an enclosure—egg, meet chicken; chicken, meet egg.

---

# Factoring out independent code

`XMLPost`:

- tightly couples _data types_ to _parsing strategies_

- introduces margin for error

^ There are several ways to solve these problems. We could add a subclass of `Post` which wraps an `XMLPost`, adding an enclosure; we could add an `isPodcast` flag to `Post` or `XMLPost`; we could add an optional enclosure property to `Post` or `XMLPost` and have the appropriate views/controllers check for its presence. But these are all working around a problem with the class hierarchy itself: it’s brittle, saying both more and less than what we mean. How so?

^ It says _more_ than what we mean in that it’s too specific. By encoding the parsing strategy in the class hierarchy, we remove our ability to vary it independently of the leaf nodes.

^ It says _less_ than what we mean in that it’s too general. `XMLPost` is abstract, and thus it wouldn’t work for it to be initialized directly; only its subclasses should be. But this isn’t directly expressible in the language, so some tired or unwary developer may make this mistake at some point.

^ Fortunately for us and our somewhat contrived example, a better factoring is straightforward too.

---

# Factoring out independent code

```swift
struct XMLParser { … }

class RSS1Post: Post { … }
class RSS2Post: Post {
	init(data: NSData) {
		let parser = XMLParser(data)
		super.init(title: parser.evaluateXPath(…), …)
	}
}
```

^ Now the various posts all just subclass `Post` directly, while `XMLParser` is completely independent and doesn’t need to know anything about `Post` at all.

^ By factoring parsing out of the `Post` hierarchy, we can vary it independently of the RSS & Atom post classes. We may still have work to do if a replacement for or addition to our parsing strategy doesn’t use the same interface, but by factoring it out in the first place we are free to abstract the variable concept of _parsing_ separately from the orthogonal concept of _data_.

^ Obviously, this is a contrived example; we all know better than to implement XML parsing in our model classes. (Right?) But I know I can think of examples where I’ve conflated the concerns of one class hierarchy with an orthogonal set which _happened_ to align—at least at the moment. It’s an easy enough mistake to make, and correcting it proactively like this helps us make our code more flexible to change, bending with it rather than breaking.

^ Even better, doing this setup makes it much easier for us to apply other approaches to reducing brittleness. We didn’t need `XMLPost`; do we need `Post`?

---

# Approach 2:
# Protocols, not superclasses

^ In addition to sharing implementations, we often employ subclasses in order to share an interface between distinct implementations—as with `Post` and its subclasses.

^ Superclasses of this nature are often (mostly) abstract. Why do we care whether we use a class interface for this? After all, if a superclass is abstract, then its subclasses aren’t tightly coupled to its implementation details, right?

^ Well, any subclass is coupled to at least _one_ implementation detail of its superclass: which class that even _is_. When a method takes a parameter whose type is of a specific class, it’s almost always overconstraining—tightly coupling. We don’t (and shouldn’t) care that we receive an instance with that specific class’ memory layout and implementation; we care that it has a specific interface. (Obviously, if we _did_ care about the memory layout, the two classes would be tightly coupled by definition.)

^ This could also needlessly force consumers of our API to jump through hoops when it would be more convenient, more elegant, or more efficient for them to use some other type to implement the interface.

^ Protocols are a way to have our cake and eat it too: we can express the interface we need to interact with _precisely_, and _without_ introducing tight coupling to irrelevant implementation details like what specific class it may be.

---

# Protocols are interfaces

- specify required properties & methods

- Cocoa protocols:

	- delegate/data source protocols (e.g. `UITableViewDelegate`)

	- model protocols (e.g. `NSDraggingInfo`, `NSFetchedResultsSectionInfo`, `NSFilePresenter`)

	- behaviour protocols (e.g. `NSCoding`, `NSCopying`)

^ At the core, protocols are just interfaces, somewhat analogous to a purely abstract class. A protocol specifies required properties & methods. A class which conforms to a protocol must implement all of the required properties & methods in order to compile.

^ Cocoa’s use of protocols can, broadly, be broken down into three categories:

^ First, protocols which delegate some of an object’s behaviour to some other object. `UITableViewDelegate` and `UITableViewDataSource` are examples of this kind of protocol.

^ Second, protocols which resemble a model object, combining a few properties and perhaps some methods around a single theme. This is a little vague, and not exactly common in Cocoa. `NSFilePresenter` might be an example: it combines a presented item’s URL and operation queue with behaviours relating to serialized access to and changes of the file being presented.

^ Cocoa also uses this kind of protocol in cases where the implementor appears to want to elide specific type information. For example, we don’t know what particular class is going to be passed to a method receiving `NSDraggingInfo` or `NSFetchedResultsSectionInfo`, which means Cocoa avoids vending implementation details via its types, and further avoids compatibility issues when they later change the underlying implementations.

^ Third, protocols which describe a single behaviour which an object must be able to perform; for example, conforming to `NSCoding` means that instances of a class can be encoded/decoded; conforming to `NSCopying` means that they can be copied. In Cocoa these typically end in -ing (`NSCopying`, `NSCoding`, `NSLocking`), whereas in Swift’s standard library these typically end in -able (`Equatable`, `Comparable`, `Hashable`).

^ Note that all of these are still just interfaces: they could have used abstract classes instead, but that would constrain the concrete implementations to a specific class hierarchy, which would make composing them inconvenient in many cases.

^ So if protocols are shared interfaces, how might we use them to share the key interface in the data model of our contrived aggregator/bookmarking app?

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
class RSS2Post: Post {
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
protocol PostType {
	var title: String { get }
	var author: String { get }
	var postedAt: NSDate { get }
	var URL: NSURL { get }
}

struct RSS2Post: PostType {
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

^ I’ve elided the other post types, but they undergo analogous changes to those made to `RSS2Post`. `Post` is changed from a class to a protocol definition, `PostType`; `RSS2Post` no longer needs to be a `class` at all, so we can use a value type. It has constants for its title, &c.; and it binds values to these instead of calling into a superclass’ initializer.

^ Note that in Swift, subclassing and conformance to a protocol use the same syntax; this is not a typo! We’re still conveying the same “is a” relationship as we were, but now we’re doing so without coupling `RSS2Post` to any particular implementation.

^ `PostType` is pretty minimalistic now; it’s a model protocol, but it doesn’t conflate its purpose as a data model with any orthogonal concerns of parsing or presentation. There may not be any need to factor it out further, but what about the other protocols we may have written in our apps: delegate protocols?

---

# Factor your protocols, too

- `UITableViewDelegate` API ref is in _**9 sections**_

- `UITableViewDataSource` & `UITableViewDelegate` aren’t _really_ independent

- exact same problem as with ill-factored classes

- factor around independent concepts instead

^ Delegate protocols have a tendency to grow over time. For example, have you ever written a class implementing every single method in `UITableViewDelegate`?

^ The API ref for `UITableViewDelegate` is broken into _9 sections_, but by my count it’s more like thirteen different responsibilities including display notifications, selection, editing, and layout—which strays dangerously near to a view responsibility—`UITableViewDataSource` territory.

^ And on that note, not only is `UITableViewDelegate` massive, it’s almost inextricably intertwined with `UITableViewDataSource`. How many people have ever written a class conforming to either `UITableViewDelegate` _or_ `UITableViewDataSource`, but not _both_?

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

^ As a low-effort, medium-reward measure, you can start by adding properties for the callbacks, or by splitting e.g. menu/editing interactions off into smaller purpose-specific delegates. If the consumer chooses to implement them all with the same object, they still can; but they’re no longer prevented from using a factoring more appropriate to their application’s design.

^ Note that you can also provide public implementations of these protocols for the default behaviours; this can make it easy for consumers to wrap or otherwise compose them, making the class more convenient to use, more flexible, and simpler to write. It’s easier to understand, as well, since we’ve encapsulated—and documented!—the distinct roles of these interfaces in the API, instead of lumping them in with everything else in the kitchen sink of the delegate protocol.

^ Even better, once we’re using model protocols, we can employ our next approach to help us compose them.

---

# Approach 3:
# Minimize interfaces with functions

^ Swift’s functions offer overloading, generic functions, and simple, powerful function types; we’ll start with overloading.

---

# Function overloading is almost an interface

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

^ Swift supports multiple dispatch: which function will be executed when you call a function can depend on both the argument and return types.

^ That means that _free_ (i.e. ordinary) functions can act a lot like methods: you can write functions `first(Stream)` and `dropFirst(Stream)` taking `Stream` and another pair by the same names taking `List`.

^ Now we can call `first()` and pass in either a `Stream` or a `List` and we’ll get the behaviour we want. This is almost, but not quite, enough.

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

^ Alas, that one will have to wait for the Swift team’s memoirs. We can assume that they have their reasons, though—and so might we. After all, sometimes you need just a _little_ extra functionality than just a bare function type.

---

# Approach 4:
# Abstract into (many) minimal types

^ Similarly, it’s reasonable to ask yourself just how much you need to abstract. Does your `Author` type really need a `Bibliography` instead of a list of publications?

^ Minimal types are ones where you can’t really factor them out any further.

---

# Lists as a protocol

^ For example, we could describe lists using a protocol, like this one which we saw earlier:

```swift
protocol ListType {
	typealias Element
	init(first: Element, rest: Self?)
	var first: Element { get }
	var rest: Self?
}
```

How many different implementations of lists do we need, exactly?

^ But doing so raises a question: Do we even need to?

---

# Lists as a minimal type

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

Use `enum` for fixed sets of alternatives:

```swift
enum Result<T> {
	case Success(Box<T>)
	case Failure(NSError)
}
```

^ An `enum` provides a fixed set of cases, which provide the alternative values for the type.

^ For example, here we have `Result`. `Result` is a good return type for operations which can end either in a successful value or an error.

^ This is a particularly good use case for `enum` since there are only two possibilities: it either worked or it didn’t. We don’t need to worry about adding more cases later on and having to update every function using `Result` to match.

^ On the other hand, if the set of cases is open-ended, we might want to use a protocol instead. Our aggregator/bookmarking app’s `Post` model types are exactly such a case—adding support for podcasts to a `Post` `enum` could require traipsing all across the project.

^ There are lots of other minimal types we could be enjoying: `List`, `Result`, `Box`, `Either`, `Stream`, and so forth are all common types we could use in our apps. There are others capturing similarly minimal patterns; and which we employ will depend heavily on what our app is doing. But we can always approach the task at hand by looking for minimal abstractions which will express it more elegantly. I highly recommend doing so.

^ Each of these approaches will help us avoid subclassing when we’re working purely with our own code, but as often as not, we’re using Apple’s, too. What about when we’re dealing with Cocoa?

---

# Caveat: Cocoa _requires_ you to subclass

^ Many Cocoa classes that we use to form the skeleton of our apps are designed to be subclassed for common uses. `UIViewController` and `UIView` are two of the most common examples, but they’re hardly the only ones. What do we do in these situations?

---

# Write minimal subclasses

- can you configure an instance instead of subclassing?

- extract distinct responsibilities into their own types

- code defensively

^ We may be required to subclass, but that doesn’t mean game over. While we may still be at the whim of new SDKs when it comes to unanticipated change in Apple’s classes, we can defend against the worst.

^ First off, ask yourself if you can configure an instance instead of subclassing. Sometimes classes like `UIViewController` which _typically_ require subclassing can simply be set up instead—whether in a xib, a storyboard, or in code.

^ Sometimes the answer to this question is going to be “no.” Likewise, sometimes jumping through hoops to avoid a subclass outright won’t be worth it.

^ In those cases we can apply the same approaches we’ve considered already. For example, we would want to make sure that distinct responsibilities are being handled by distinct types. By factoring responsibilities out of the subclass, we avoid making assumptions about the superclass. After all, no SDK change will ever invalidate an assumption you haven’t made.

^ You can think of this as coding defensively. In Objective-C, best practice for categories on another party’s types—whether Apple’s or a third-party—is to prefix the method names so as to avoid collisions.

^ Likewise, you can insulate your code from future change—and future coupling!—by minimizing the interfaces that your code operates on, and therefore assumes.

^ As with everything else we’ve discussed, this is a matter of discipline: good habits make better code. It’s the same way with making sure we don’t couple too tightly to our own classes, subclass or otherwise.

---

# A `final` piece of advice

- make all classes `final` by default

- only remove `final` as a conscious choice

- consider leaving a comment as to why you did

^ To that end, I recommend making all classes `final`—which means “this class cannot be subclassed”—by default.

^ Note that I say “by default”—these approaches are tradeoffs, and you may find that this is the wrong one for your specific case. However, by always _defaulting_ to `final`, you ensure that any time you’re removing the keyword, it’s as a conscious decision in light of the circumstances you’re designing for.

^ Further, if you leave a comment as to _why_ the class isn’t `final`, you’ll inform your teammates (and your future self!) of the reasoning behind the decision. Compromise for a deadline is a valid reason, but the reminder can help you to keep in mind that tightly coupling to the superclass should likely still be avoided.

---

# Takeaway

- subclassing is for the weak and timid
- reuse interfaces with protocols
- reuse implementations by factoring & composing

^ The takeaway is pretty simple: subclassing is a weak and brittle substitute for better engineering practices: protocols, minimal types, factoring, and composition.

^ So don’t subclass.

---

> Thanks to Matt Diephouse, Ken Ferry, Kris Markel, Andy Matuschak, Ryan McCuaig, Kelly Rix, Haleigh Sheehan, Justin Spahr-Summers, Patrick Thomson, and you ❤️
