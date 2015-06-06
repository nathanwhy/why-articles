
Demystifying Retain Cycles in ARC

揭秘 ARC 下的循环引用

原文：http://digitalleaves.com/blog/2015/05/demystifying-retain-cycles-in-arc/

Retain cycles in ARC are kind of like a Japanese B-horror movie. When you start as a Cocoa/Cocoa Touch developer, you don’t even bother about their existence. Then one day one of your apps starts crashing because of memory leaks and suddenly you become aware of them, and start seeing retain cycles like ghosts everywhere. As the years pass, you learn to live with them, detect them and avoid them… but the final scare is still there, looking for its chance to creep in.

ARC 下的循环引用类似于日本的 b-horror 电影。当你刚成为苹果开发者，你或许不会因为他们的存在而操心。等到某天，你的一个 app 因内存泄露而闪退，你会突然感受到他们幽灵般的存在。随着时间的进展，你开始学会检测和避免它们，但是担心它们可能已经悄然发生的恐惧仍然存在。

One of the biggest disappointments of ARC for many developers (including me) was that Apple kept ARC for memory management. ARC unfortunately doesn’t include a circular reference detector, so it’s prone to retain cycles, thus forcing the developers to take special precautions when writing code.

ARC 令许多开发者最失望的地方在于苹果一直未公开 ARC 的内存管理。ARC 很不幸地没有包括循环引用检测，因此很容易就产生循环引用，迫使开发者需要在写代码的时候采取一些防范措施。

Retain cycles are a obscure topic for iOS developers. There are lot of misinformation in the web [1][2], to the point of people giving wrong suggestions and “fixes” that could even potentially lead to problems and crashes in your App. In this article, I would like to shed some light on the subject matter.

循环引用一直是一些 iOS 开发者感到费解的一个课题。
网上对此也有一些误解[[1]](https://dhoerl.wordpress.com/2013/04/23/i-finally-figured-out-weakself-and-strongself/)[[2]](http://www.reigndesign.com/blog/debugging-retain-cycles-in-objective-c-four-likely-culprits/)，这些文章给了错误的建议和修复方法，其方法甚至可能潜在引发问题和导致 app 闪退。在这片文章，我想要针对这些问题解释清楚。

##Some (brief) theory
##理论简介
Memory management in Cocoa dates back to Manual Retain Release (MRR). In MRR, the developer would declare that an object must be kept in memory by claiming ownership on every object created, and relinquishing it when the object was not needed anymore. MRR implemented this ownership scheme by a reference counting system, in which each object would be assigned a counter indicating how many times it has been “owned”, increasing the counter by one, and decreasing it by one each time it was released. The object would then cease to exist when its reference counter reached zero. Having to take care of reference counting manually was really an annoyance for the developer, so Apple developed ARC (Automated Reference Counting) to free the developers of having to manually add the retain and release instructions, allowing them to focus in the problem being solved by the App. Under ARC, a developer will define a variable as “strong” or “weak”. A weak App will not be retained by the object declaring it, whereas a strong declaration will retain the object and increment its reference count.

内存管理可以追溯到手动内存管理（Manual Retain Release，简称 MRR）。在 MRR，开发者创建的每一个对象，需要声明其拥有权，从而保持对象存在于内存中，当对象不再需要的时候撤销拥有权释放它。MRR 通过引用计数系统实现这套拥有权体系，也就是说每个对象有个计数器，通过计数加1表明被一个对象拥有，减1表明不再持有。当计数为零，对象将被释放。由于手动管理内存实在太烦人，因此苹果推出了自动引用计数（ARC）来解放开发者，不再需要开发者手动添加 retain 和 release 操作，从而可以专注于 App 开发。在 ARC，开发者将会定义一个变量为“strong”或“weak”。一个 weak 弱引用无法 retain 对象，而 strong 引用会 retain 这个对象，并将其引用计数加一。

![retainCycle](retainCycle.jpg)

##Why should I care?
##我应该关心什么？

The problem with ARC is that it’s prone to retain cycles. A retain cycle occurs when two different objects contain a strong reference to each other. Think of a Book object that contains a collection of Page objects, and that every Page object has a property pointing to the book the page is contained in. When you release the variables that point to Book and Page, they still have strong references among them, so they won’t be released and their memory freed, even though there are no variables pointing to them.

在 ARC 需要注意的问题是循环引用。当两个不同的对象各有一个强引用指向对方，那么循环引用便产生了。试想下，一个 *book* 对象持有多个 page 对象，每个 page 对象又有个属性指向它所属的 *book* 对象。当你释放了持有 book 和 page 对象的变量时，他们仍然还有强引用指向各自，因此你无法释放他们的内存，即使已经没有变量持有他们。

Unfortunately, not all retain cycles are so easy to spot. Transitive relationships between objects (A references B, which in turn references C, which happens to reference A) can lead to retain cycles. To make things worse, both Objective-C blocks and Swift closures are considered independent memory objects, so any reference to an object inside a block or closure will retain its variable, thus leading to a potential retain cycle if the object also retains the block.

不幸的是，循环引用在实际中并没有那么容易被发现。多个对象之间（A 持有 B，B 持有 C，C 也恰好持有 A）也可以产生循环引用。更糟的是，Objective-C block 和 Swift 闭包都是独立内存对象，它们会持有其所引用的对象，于是就引发了潜在的循环引用问题。

Retain cycles can be potentially dangerous for an App, leading to high memory consumption, bad performance and crashes. However, there is no clear documentation from Apple about the different scenarios in which a retain cycle may occur, and how to avoid them. This has lead to a number of misconceptions and bad programming practices.

循环引用对 app 有潜在的危害，会使内存消耗过高，性能变差和 app 闪退等。然而，苹果文档对于可能发生循环引用的场景以及如何避免并没有详细描述，这就容易导致一些误解和不良的编程习惯。

##Use case scenarios
##发生场景
So, without further ado, let’s analyze some scenarios to determine whether or not they cause a retain cycle and how to avoid them:

废话不多说，我们一起来分析可能产生循环引用的场景，以及如何避免它。

###Parent-child object relationship
###父子对象关系

This is the classic example of retain cycle, and unfortunately, the only one addressed by the Apple documentation. It’s the example of the Book and the Page objects I described before. The classic solution for this situation is defining the variable representing the parent in the Child class as weak, thus avoiding the retain cycle

父子对象关系是一个循环引用的典型案例，不幸的是，它也是唯一一个存在于苹果文档中的案例。其实就是前文描述的 Book 与 Page 案例。典型的解决方法就是，在子类定义一个指向父类的变量，声明为 weak 弱引用，从而避免循环引用。

```swift
class Parent {
   var name: String
   var child: Child?
   init(name: String) {
      self.name = name
   }
}
class Child {
   var name: String
   weak var parent: Parent!
   init(name: String, parent: Parent) {
      self.name = name
      self.parent = parent
   }
}
```

The fact that parent is a weak variable in Child forces us to define it as an optional type in swift. An alternative for not having to use an optional is declaring parent as “unowned” (meaning that we don’t claim ownership or memory management on the variable). However, in this case we must be extremely careful to ensure that Parent doesn’t become nil as long as there is a Child instance pointing to it, or we will end up with a nasty crash:

事实上在 swift，子类指向父对象是一个弱引用，这就迫使我们将该弱引用定义为 optional 类型。如果不使用 optional 可以有另一种做法，将指向父对象的变量声明为“无主引用（unowned）”（表明我们不持有该对象，也不对其进行内存管理）。然而在这个案例，我们必须非常关心，确保父对象不变成 nil，只要还有子对象指向它，否则会直接闪退。


```swift
class Parent {
   var name: String
   var child: Child?
   init(name: String) {
      self.name = name
   }
}
class Child {
   var name: String
   unowned var parent: Parent
   init(name: String, parent: Parent) {
      self.name = name
      self.parent = parent
   }
}
var parent: Parent! = Parent(name: "John")
var child: Child! = Child(name: "Alan", parent: parent)
parent = nil
child.parent <== possible crash here!
```

In general, the accepted practice is that the parent must own (strongly reference) its children objects, and they should only keep a weak reference to their parent. The same applies for collections, they must own the objects they contain.

通常有效的做法是，父对象必须持有（强引用）子对象，而子对象只要保持一个弱引用指向他们的父对象。这同样适用于集合对象，它们必须持有它们包含的对象。

###Blocks and closures contained in instance variables

###含有实例变量的 Block 和闭包

Another classic example, though not so intuitive. As we explained before, closures and blocks are independent memory objects, and retain the objects they reference, so if we have class with a closure variable, and that variable happens to reference a property or method in the owning object, there would be a retain cycle because the closure is “capturing” self by creating a strong reference to it.

另外一个典型的例子，可能不是那么直观。如我们前面解释的，闭包和 block 都是独立的内存对象，会 retain 它们所引用的对象，因此如果我们有个类，里面有个闭包变量，并且这个闭包恰好引用了自身所属对象的一个属性或方法，那么就可能产生循环引用，因为闭包会创建强引用捕获“self”。

```swift
class MyClass {
   lazy var myClosureVar = {
      self.doSomething()
   }
}
```

The solution in this case implies defining a “weak” version of self and sending that weak reference to the closure or block. In Objective-C we will define a new variable:

这个案例的解决方法是定义一个弱版本的 self，然后在闭包或 block 中使用。在 objective-C,我们会定义一个新的变量：

```objc
- (id) init() {
   __weak MyClass * weakSelf = self;
   self.myClosureVar = ^{
      [weakSelf doSomething];
   }
}
```

Whereas in Swift we just need to specify the “[weak self] in” as the closure starting parameter:

然而在 Swift 我们只需要在闭包的头部声明 "[weak self in]":

```swift
var myClosureVar = {
   [weak self] in
   self?.doSomething()
}
```

This way, when the closures reaches the end, the self variable is not strongly retained, so it gets released, breaking the cycle. Note how self becomes an optional inside the closure when declared weak.

在这个方法，当闭包结束的时候，内部的 self 变量不会被强引用，所以它会被释放，破除了循环引用。注意当 self 被声明为 weak，闭包内部的 self 是个可选值。

###GCD: dispatch_async

Contrary to what’s usually believed, dispatch_async per se will NOT cause a retain cycle

和我们通常所认为的不同，dispatch_async 自身不会造成循环引用

```swift
dispatch_async(queue, { () -> Void in
   self.doSomething(); 
});
```

Here, the closure has a strong reference to self, but the instance of the class (self) does not have any strong reference to the closure, so as soon as the closure ends, it will be released, and so no cycle will be created. However, sometimes it’s (incorrectly) assumed that this situation will lead to a retain cycle. Some developers even get to the point of recommending all references to “self” inside a block or closure to be weak:

在这里，闭包会强引用 self，但是实例化的 self 不会强引用闭包，所以一旦闭包结束，它就会被释放，所以循环引用也不会产生。然而，总有些开发者认为它可能会产生循环引用。有些开发者甚至以为，所有在 block 和闭包里面的 self 都需要弱引用：

```swift
dispatch_async(queue, {
   [weak self] in
   self?.doSomething()
})
```

This is not a good practice for every single case in my opinion. Let’s suppose that we have an object that launches a long background task (like downloading some content from the network), and then invokes a method of “self”. If we pass a weak reference to self, our class could end its lifecycle and get released before the closure ends, so when it invokes the doSomething() method, our class instance doesn’t exist anymore, so the method never gets executed. The proposed solution (by Apple) for this situation is declaring a strong reference to the weak reference (???) inside the closure:

在我看来，每种情况都采用这种方法并不是一个好的实践。让我们试想下，如果我们有个对象，用于发送一个后台任务（比如下载数据），并且调用了 self 的一个方法。这时如果我们弱引用 self，该对象的生命周期结束早于闭包结束被释放，因而当我们的闭包调用的 doSomething()方法，该对象可能就不存在了，方法也得不到执行。合适的解决方法是（苹果推荐）在闭包内部，声明一个强引用指向弱引用。

```swift
dispatch_async(queue, {
   [weak self] in
   if let strongSelf = self {
      strongSelf.doSomething()
   }
})
```

Not only do I find this syntax disgusting, unintuitive and tedious, but I think it defeats the entire purpose of having closures as independent processing entities. I think it’s much better to understand the lifecycle of our objects and know exactly when should we declare a weak inner version of our instance and which are the implications for the lifetime of our objects. But then again, this is something that distracts me from the problem my App is solving, it’s code that wouldn’t need to be written if Cocoa didn’t use ARC.

我觉得这种语法不仅恶心乏味不直观，而且违反了闭包作为一个独立处理实体的原则。学会理解对象的生命周期，明白何时应该声明弱引用，以及对象生存周期的意义，这很重要。但是，这又使得我分心而无法专注于 app 开发的问题本身，如果 Cocoa 不使用 ARC，也就不必要写这些代码。

###Local closures and blocks
###本地闭包和 block

Closures and blocks local to a function, not referenced or contained in any instance or class variable will NOT cause a retain cycle per se. One of the common examples of this is the animateWithDuration method of UIView:

函数的闭包和 block 如果没有引用任何实例或类变量，其本身也不会造成循环引用。最常见的一个例子就是 `UIView` 的 `animateWithDuration`。 

```swift
func myMethod() {
   ...
   UIView.animateWithDuration(0.5, animations: { () -> Void in
      self.someOutlet.alpha = 1.0
      self.someMethod()
   })
}
```

As with dispatch_async and other GCD related methods, we should not worry about retain cycles in local closures and blocks not referenced strongly by the class instance.

和 dispatch_async 和其他相关的 GCD 相关方法一样，我们不需要担心局部变量闭包和 block 被类实例强引用而产生循环引用。

###Delegation scheme
###代理协议

The delegation scheme is one of the typical scenarios where you want to use a weak reference to avoid a retain cycle. It is always a good practice (and generally safe) to declare the delegate as weak. In Objective-C:

代理协议也是一个典型的场景，需要你使用弱引用来避免循环引用。将代理声明为 weak 是一个即好又安全的做法：

`@property (nonatomic, weak) id <MyCustomDelegate> delegate;`

And in swift:
在 swift：

`weak var delegate: MyCustomDelegate?`

In most cases, the delegate of an object has instantiated the object or is supposed to outlast the object (and react to the delegate methods), so under a good class design we should not find any problems regarding the lifetime of objects.

在大多数的例子，一个对象的代理持有一个实例化的对象，或应当生命周期长于该对象（从而响应代理方法），因此在一个优秀的类的设计，我们不应该找到任何无视对象生命周期的问题。

###Debugging retain cycles with Instruments.
###使用工具调试循环引用

It doesn’t matter how hard I try to avoid retain cycles, sooner or later I forget to include a weak reference and accidentally create one (thanks for nothing, ARC!). Luckily, the Instruments application included in the XCode suite is a great tool for detecting and locating retain cycles. It is always a good practice to profile your App once the development phase is over and before submitting it to the Apple Store. Instruments has many templates for profiling different aspects of the App, but the one we’re interested in is the “Leaks” option.

不管我多努力仔细，我有时还是会忘记声明一个弱引用，然后意外地创建一个新的对象（ARC 也就那样）。幸运的是，XCode 自带了一个很强大的工具，用于检测和定位循环引用。一旦你的 app 开发结束，即将提交到 Apple Store，先分析你的 app 是一个好的习惯。工具有很多组件，可以用来分析 app 的不同方面，但是我们现在关心的时 Leak 选项。

![instrument](instruments.png)

Once Instruments opens, you should start your application and do some interactions, specially in the areas or view controllers you want to test. Any detected leak will appear as a red line in the “Leaks” section. The assistant view includes an area where Instruments will show you the stack trace involved in the leak, giving you insights of where the problem could be and even allowing you to navigate directly to the offending code.

一旦打开工具，你应该启动你的应用，执行一些交互操作，特别是你想要测试的区域或视图控制器。被检测到的泄露都会以一条红色线显示在 Leaks 区域。assistant view 会显示关于泄露的栈追踪，甚至可以直接定位到出问题的代码。

![instrument2](instruments2.png)

