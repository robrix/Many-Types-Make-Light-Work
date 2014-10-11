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

# Step 1.:
# factor _ruthlessly_

^ Chances are your classes are too big. Follow the One Responsibility Rule: break them down into one class per responsibility and use those together instead.

^ Factoring helps us reduce subclassing most when we’re subclassing in order to share an implementation. For example, `UIView` and `NSView` implement common behaviours which many views will need, e.g. drawing, converting geometry between coordinate systems, animation, appearance lifecycle, event handling; a better factoring of these would enable us to use precisely the behaviours we want without subclassing & potentially gaining undesirable behaviours as well.

^ Further, we’d be that much better insulated by default from unanticipated change in Apple’s frameworks.

^ This sort of subclassing is common within our own codebases as well, e.g. model classes often form hierarchies of this nature. Split them up & factor them out; the resulting flexibility often ends up reducing the size of the codebase (and thus surface area for bugs) as well.

^ Implementation sharing is not the only reason we subclass, though, so other approaches are needed too.

---

# Approach 2.:
# `protocol`

^ Sometimes you need to be able to describe different types as being related—they need to share an interface.

---

# Approach 3.:
# `enum` and `switch`

---

Exception: Many Cocoa [Touch] classes are designed to be subclassed.

---

Solution: Write a composable subclass.

---

Example: `{NS,UI}ViewController`.

---

# Thanks

- Matt Diephouse
- Kris Markel
- Ryan McCuaig
- you ❤️

![](http://upload.wikimedia.org/wikipedia/commons/d/db/Railroad_Coupling_(CMRR\).jpg)
