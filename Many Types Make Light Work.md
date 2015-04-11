# Many Types Make Light Work[^1]

### 🐦 [@rob_rix](https://twitter.com/rob_rix)<br>🐙 [@robrix](https://github.com/robrix)<br>📩 [rob.rix@github.com](mailto:rob.rix@github.com)

^ who am I

^ where do I work

^ what am I going to talk about

[^1]: https://github.com/robrix/Many-Types-Make-Light-Work

---

# More code = more complexity = more bugs

Code reuse reduces risk.

- implementations:

	- Don’t Repeat Yourself (DRY)

	- functions, methods, (sub)classes, &c.

- interfaces:

	- use the same code with different types

	- subclassing, protocols

^ adding code increases complexity & therefore risk—more bugs to fix, more change, turtles all the way down.

^ Code reuse limits complexity. “Code reuse” means:

^ Reusing implementations boils down to “Don’t Repeat Yourself” or “DRY.” We factor out code we want to reuse into a function, method, or class, and call it from the places which need its behaviour. One function, used by multiple callers.

^ The flip side is reusing interfaces. A function which takes a parameter of a given class can take any subclass, because a class defines an interface shared—reused—by its subclasses. Likewise, a function which takes a parameter of a given protocol can take any concrete type conforming to it—that’s what protocols are for. One function, taking multiple types.

^ Broadly speaking, the most common tools we employ for code reuse are subclassing & composition.

---

# Subclassing

- inherit superclass’ interface & implementation

- describes the class hierarchy at compile time

# Composition

- composition is _~~when you put a thing in a thing and then it’s a thing then~~_ combining code

- describes the runtime object graph/call stack

^ Subclassing is immediately familiar: a subclass inherits its superclass’ interface & implementation. That makes subclassing convenient both for grouping similar things together under a shared interface, and for sharing the superclass’ functionality between multiple subclasses.

^ Composition may be a less familiar term, but you use the concept all the time: it boils down to objects which can be used together. For example, a `UIView` is composed with its superview, its subviews, and its layer; a `UIViewController` is composed with its parent controller, its child controllers, and its views; and so on.

^ We reuse implementations using composition by simply using the different implementations together. Composition doesn’t provide interface reuse—we’d describe that as the job of its fraternal twin, abstraction—but it does work just fine with shared interfaces: anything which composes with a given interface can compose with any type providing it.

^ But even though we _can_ use composition & abstraction to reuse implementations & interfaces, we will often reach for subclassing first.

^ However…

---


> subclassing ≃ 💥🔥💀

^ …subclassing tends to cause us problems. For example…

---

> Subclassing _conflates_ interface & implementation reuse

^ The perceived convenience of subclassing comes at a cost: if we want to reuse the interface, or just part of the implementation, the rest of the implementation tags along anyway.

---

> Subclassing _couples_ subclasses to their superclass

^ This means that every change to the superclass affects each subclass. If a change invalidates some assumption of a subclass, that subclass now has a bug from a change in another piece of code. Likewise, if the superclass calls its own methods (as they tend to), the subclass can also invalidate an assumption of the superclass—even if that assumption is new.

^ For example, on OS X Mavericks, `NSViewController` doesn’t have the convenient `-viewWillAppear`, `-viewDidAppear`, etc. methods which we’re familiar with from `UIViewController`, so it was common to implement those methods in a subclass and call them at the appropriate times.

^ Under Yosemite, `NSViewController` adds and calls those methods, meaning we now have a bug. These methods are called twice: once by our code, and once by our superclass. All we did is compile against the new SDK.

---

> Subclassing _enables_ tight coupling in composed code

^ Subclassing also enables other code using the hierarchy to make more assumptions about subclasses than would otherwise be possible, simply because the interfaces are broader than they need to be—and they get broader with each layer of subclass. This can lead to even more coupling and brittleness, unintentionally increasing the risk and cost of change (whether we make the change or Apple does).

^ The good news is that since composition/abstraction solve the same problems as subclassing, we have a simple solution to this kind of problem:

---

> Don’t subclass.
— me, here, now

^ Subclassing is _unnecessary_. Composition lets us have our cake and eat it too: we can reuse interfaces and implementations at our discretion without automatically coupling tightly.

^ It’s still possible for us to write tightly coupled code _without_ subclassing, of course, but it’s less automatic, and decoupling is easier.

^ We’ve been trained to subclass by our peers, mentors, books, blog posts, code bases, and by the frameworks and languages themselves. But it doesn’t have to be that way, and Swift makes _not_ subclassing easier than ever.

^ To that end, we’re going to look at some approaches to writing  more flexible, reliable, & maintainable code by not subclassing. While these are presented separately, they aren’t mutually exclusive; you can mix and match to fit the task at hand.

---

# Approach 1:
# Factor class hierarchies out

^ In the Gang of Four’s _Design Patterns_, the authors describe the implementation of a hypothetical WYSIWYG editor.

^ They use a wonderful phrase to describe how they support a stable `Window` abstraction across multiple platforms which each have different ideas of what a window is:

---

> Encapsulate the concept that varies.
— Gamma, Helm, Johnson, & Vlissides’ _Design Patterns_

^ Encapsulating—_factoring out_—the thing which we wish to be able to vary is the key here. Cross-platform code reuse may not currently be in vogue, but the principle is the same.

^ As an example of this, let’s consider the data model for a hypothetical aggregating/bookmarking app. It already manages tweets, and now we want to add RSS to it as well.

---

# Factor out independent work

```swift
class Post {
	let title: String
	…
}

class Tweet: Post { … }

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

^ But the _next_ time we want to change this code, we face substantial risk. Bugs or features that require us to change the parsing strategy will incur a lot of risk for `XMLPost` _as well as each subclass_.

---

# Factoring out independent work

`XMLPost`:

- tightly couples _data types_ to _parsing strategies_

- introduces margin for error

- is inflexible to change

^ The `XMLPost` class hierarchy itself is the problem.

^ It says _more_ than what we mean in that it’s too specific. By encoding the parsing strategy in the class hierarchy, we limit our ability to vary it independently of the leaf nodes.

^ It says _less_ than what we mean in that it’s too general. `XMLPost` is abstract, and thus we shouldn’t be able to instantiate it directly. But this isn’t directly expressible in the language, so some tired or unwary developer is likely to make this mistake at some point.

^ Fortunately, a better factoring is straightforward too.

---

# Factoring out independent work

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

^ Now the RSS posts subclass `Post` directly, while `XMLParser` is completely independent and doesn’t need to know anything about `Post` at all.

^ The post classes still depend on the parser, so changes to it (especially its interface) can still affect them, but _only at the points in which they actually interact with it_—the interface.

^ We’re free to vary parsing within that interface, now, because we’ve encapsulated the varying concept of _parsing_ orthogonally to the static concept of _data_.

^ Obviously, this is a contrived example; we all know better than to implement XML parsing in our model classes. (Right?) But I know I can think of examples where I conflated the concerns of one class hierarchy with an orthogonal set of concerns which _happened_ to align, at least at the moment.

^ It’s an easy enough mistake to make, so maybe a rule of thumb would help:

---

> Favour solutions which simplify the code base.

^ All else being equal, favour solutions which result in a simpler code base. That means factoring: smaller components make for a simpler code base.

^ With `XMLPost` factored away, it’s much easier for us to apply other approaches to reducing brittleness. We didn’t need `XMLPost`; do we need `Post`?

---

# Approach 2:
# Protocols, not superclasses

^ While `XMLPost` shared an implementation, `Post` shares an interface: consumers taking a `Post` can accept any subclass and access its `title`, &c.

^ Unfortunately, even subclassing an abstract class like `Post` couples to at least _one_ implementation detail of its superclass: which class that even is.

^ A function taking a specific class is usually overspecifying. It doesn’t care about the memory layout or implementation details, just the methods/properties it can access—the interface.

^ At the same time, it also needlessly forces consumers to jump through hoops when it might be more convenient, elegant, or efficient to use some other type which can’t subclass the abstract superclass.

^ Protocols give us the best of both worlds: we can express precisely the interface we need, without coupling to or burdening consumers with irrelevant implementation details.

---

# Protocols are interfaces

- just the _relevant_ details: properties & methods

- Cocoa protocols:

  1. behaviour: `NSCoding`, `NSCopying`, `UITableViewDelegate`
  2. model: `NSFetchedResultsSectionInfo`, `NSFilePresenter`

^ Protocols are just interfaces, specifying properties & methods which conforming types must provide.

^ Cocoa’s protocols fall into two broad categories:

^ First, protocols which describe a set of behaviours. For example, `NSCoding` describes encoding/decoding. In Cocoa, simple behaviour protocols end in -ing (`NSCopying`, `NSCoding`, `NSLocking`). In Swift’s standard library, they end in -able (`Equatable`, `Comparable`, `Hashable`).

^ Delegate & data source protocols are also examples of this sort, although usually much larger.

^ Second, protocols which resemble a model object, combining a few properties and perhaps some methods around a single theme. `NSFilePresenter` is one example, providing a presented URL & operation queue as well as behaviours around the presented file.

^ Both categories are still just interfaces, and as such allow us to reuse code without constraining or coupling to implementation details.

^ Recall that post-factoring, our app has a shallow hierarchy of model classes. Since we’re no longer using subclassing for implementation sharing (aside from the storage for the properties), this turns out to be trivial to migrate to a model protocol.

---

# Using protocols to share interfaces

```swift
protocol PostType {
	var title: String { get }
	…
}

struct RSS2Post: PostType {
	let title: String
	…

	init(data: NSData) {
		let parser = XMLParser(data)
		title = parser.evaluateXPath(…)
		…
	}
}
```

^ `Post` is changed from a class to a protocol definition, `PostType`, which `RSS2Post` conforms to. Not only does it not need to subclass, it doesn’t need to be a `class` at all. It has constants for its title, &c.; and it binds values to these instead of calling into a superclass’ initializer. We can assume the other `Post` types were treated similarly.

^ Note that in Swift, subclassing and conformance to a protocol use the same syntax; this is not a typo! We’re still conveying the same “is a” relationship as we were, but now we’re doing so without coupling `RSS2Post` to any particular implementation.

^ `PostType` is a minimal model protocol, and it doesn’t conflate that responsibility with orthogonal concerns of parsing or presentation. What about behaviour protocols?

---

# Factor out independent interfaces

- `UITableViewDelegate` has >= 9 jobs (!)

- `…DataSource`/`…Delegate` are interdependent

- only used by & tightly coupled to `UITableView`

- forces implementing type to handle multiple concerns

- exact same problem as ill-factored classes

^ Delegate protocols grow over time. The API ref for `UITableViewDelegate` is broken into _nine sections_, but by my count it’s more like _thirteen_ different responsibilities including display notifications, selection, editing, and layout—which strays dangerously near to a view responsibility—`UITableViewDataSource` territory. Have you ever written a class implementing every single method in `UITableViewDelegate`?

^ Not only is `UITableViewDelegate` massive, it’s intertwined with `UITableViewDataSource`. How many people have ever written a class conforming to either `UITableViewDelegate` _or_ `UITableViewDataSource`, but not _both_?

^ Just like with classes, this is a hint that these protocols have too many responsibilities and that they haven’t been divided in the right places. Again just like with classes, factor independent concerns into separate protocols.

---

# Delegate protocols suggest better factoring

- instead, _“encapsulate the concept that varies”_

	- factor out types for independent concerns (e.g. groups)

	- KVO-compliant selected/displayed subset properties (or signals) instead of `will…`/`did…`

	- can start by splitting methods into tiny protocols

^ I’d go so far as to say that this applies to every delegate protocol. Delegating a kitchen sink of view behaviours to a single object makes it difficult to vary them independently, suggesting that both the delegate protocol _and_ the type consuming it are ill-factored.

^ Instead, we can factor them out: if a view displays elements in groups, expose an interface for the elements and for the groups. If you need callbacks for the display, selection, etc of these, expose KVO-compliant properties or signals for that state on those types. If that measures as a crucial bottleneck, give the host type properties/signals for the subset of elements in those states.

^ As a low-effort, medium-reward measure, you can start by adding properties for the callbacks, or by splitting e.g. menu/editing interactions off into purpose-specific interfaces. If a consumer chooses to implement them all with the same object, they’re free to do so; but they’re no longer _required_ to.

^ Note that you can also provide public implementations of these protocols for your default behaviours; this can make it easy for consumers to wrap or otherwise compose with them, making the class more convenient to use, more flexible, simpler to write, and easier to understand, since we’ve encapsulated—and documented!—these distinct jobs in the API.

^ Even better, once we’re using model protocols, we can employ our next approach to help us compose them.

---

# Approach 3:
# Minimize interfaces with functions

^ Every type provides some interface. Function types are particularly minimal ones. Every function type takes something (possibly `Void`), and returns something (again, possibly `Void`).

^ This means that a simple enough interface can be expressed with just a function type or two.

---

# Function types are shared interfaces

^ For example, Swift’s built-in `GeneratorOf` is a `GeneratorType`—an iterator—which you can construct with another generator, or with a function:

```swift
struct GeneratorOf<T> : GeneratorType {
	init(_ nextElement: () -> T?)

	// A convenience to wrap another GeneratorType
	init<G : GeneratorType where T == T>(_ base: G)
	…
}
```

^ `GeneratorOf` doesn’t mention the `GeneratorType` it wraps anywhere but in the type parameters of the convenience init. That tells us that it doesn’t store it in a property (since that property would need to have that type, and therefore `GeneratorOf` would need a parameter for it).

^ Instead, when you make a `GeneratorOf` wrapping another generator, it wraps it up in a closure which it passes to its other initializer; that closure has the necessary and sufficient interface shared by anything it wraps: it takes `Void` and returns an optional element.

^ If you can wrap up any function from `Void` to `Optional<T>` in a `GeneratorOf` with no other feedback provided, why does `GeneratorType` exist at all? Why don’t they _only_ use function types?

^ Alas, that one will have to wait for the Swift team’s memoirs. We can assume that they have their reasons, though—and so might we. After all, sometimes you need just a _little_ extra functionality than just a bare function type.

---

# Approach 4:
# Abstract (many) minimal types

^ This suggests that we can reuse interfaces even more simply than using protocols. For example, a minimal type which captures the entirety of a protocol may be sufficient for _every_ use.

---

# `Post` as a minimal type

```swift
struct Post {
	let title: String
	…
}

func ingestResponse(response: Response) -> Post? {
	switch response.contentType {
	case .RSS1:
		let parser = XMLParser(response.data)
		return Post(title: parser.evaluateXPath(…))
	default:
		return nil
	}
}
```

^ Case in point, we had used `PostType` to implement `Tweet`, `RSS1Post`, and `RSS2Post`, but these are practically identical. We can replace them all with a minimal `Post` model type and some smarts around ingestion.

^ Minimalism is key, here: `Post` can only be constructed with its direct fields, not directly with source data. This implies that model protocols can be captured much more easily than behaviour protocols, since the latter tend to require more choices (and are correspondingly more capable, and open-ended).

^ That’s one reason why the Swift standard library includes `GeneratorOf`, `SequenceOf`, and `SinkOf`, but not `CollectionOf` or `EquatableOf`: `CollectionType` and `Equatable` are harder to capture in a single concrete type since they require more degrees of freedom.

^ Even so, variance in a type doesn’t imply that we have to give up minimal types. It’s often a good opportunity to use an `enum`.

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

^ On the other hand, if the set of cases is open-ended, we might want to use a protocol instead.

^ There are lots of other minimal types we could be enjoying: `List`, `Result`, `Box`, `Either`, `Stream`, and so forth are all common types we could use in our apps. There are others capturing similarly minimal patterns; and which we employ will depend heavily on what our app is doing. But we can always approach the task at hand by looking for minimal abstractions which will express it more elegantly. I highly recommend doing so.

---

> Minimal types are value types

^ `Post` is a simple `struct` now. `Result` is a trivial `enum`. We could apply the same process to our `ListType` protocol to make a `List` `enum` too.

^ It’s no coincidence that each of these are value types. There are four reasons to use classes in Swift:

^ 1. Working around language flaws, e.g. by using `Box<T>`. `Box` has this covered already, though, so this doesn’t apply.

^ 2. Shared state for which copying is inappropriate, like a lock. You’ll know if you’re doing this one.

^ 3. Subclassing for code reuse, which we have better alternatives for.

^ 4. Interoperability with Objective-C; this isn’t uncommon, but it can be mitigated somewhat, as we’ll see.

^ Swift’s value types are simpler by default. Since they’re copied and not referenced, they don’t suffer aliasing or concurrency bugs. You can’t subclass them, so they present relatively low risk to change.

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

> Thanks to Matt Diephouse, Ken Ferry, Kris Markel, Andy Matuschak, Ryan McCuaig, Jamie Murai, Kelly Rix, Haleigh Sheehan, Justin Spahr-Summers, Patrick Thomson, and you ❤️
