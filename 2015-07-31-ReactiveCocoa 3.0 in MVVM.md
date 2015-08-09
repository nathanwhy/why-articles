This is my final article on ReactiveCocoa 3.0 (RAC3), where I demonstrate some more complex RAC3 usages within the context of an application built using the Model-View-ViewModel (MVVM) pattern.

这个是我最后一篇关于 ReactiveCocoa 3.0 （简称 RAC3）的文章，主要介绍了在 MVVM 实践中更多复杂的 RAC 3.0 的用法。

(译者注：前两篇为 [ReactiveCocoa 3.0 初见](http://ios.jobbole.com/82081/)和 [ReactiveCocoa 3.0 初见(2)](http://ios.jobbole.com/82125/))

ReactiveCocoa 3.0 is currently in beta, having had [four beta releases](https://github.com/ReactiveCocoa/ReactiveCocoa/releases) in the past month. Version 3.0 brings a whole new Swift API alongside an updated Objective-C counterpart. While the core concepts of functional reactive programming remain the same, the Swift API is *very* different to the versions that have come before it, using generics, custom operators and curried functions to good effect. In my previous articles I took a look at the [generic Signal class](http://blog.scottlogic.com/2015/04/24/first-look-reactive-cocoa-3.html), which allows for ‘strongly typed’ reactive pipelines, and the [SignalProducer interface](http://blog.scottlogic.com/2015/04/28/reactive-cocoa-3-continued.html), that gives a cleaner representation of signals that have side-effects.

ReactiveCocoa 3.0 当前还处于测试阶段，截止本文发表当天，已经有 [4 个 beta 版](https://github.com/ReactiveCocoa/ReactiveCocoa/releases)。相较于 Objective-C 版本，3.0 版本带来了全新的 Swift API。尽管函数响应式编程的核心理念还是保持不变的，但是 Swfit API 相比于之前的版本却有很大的不同。它使用了泛型，自定义操作符和柯里化函数，实现效果相当不错。在我前面的几个版本中，我探讨了 [generic Signal class（泛信号）](http://blog.scottlogic.com/2015/04/24/first-look-reactive-cocoa-3.html)，它允许强类型的信号管道，还有一篇 [SignalProducer interface](http://blog.scottlogic.com/2015/04/28/reactive-cocoa-3-continued.html)，是关于清除信号带来的一些副作用。

Since publishing those articles I’ve had quite a few people ask me to demonstrate some more complex RAC3 code, hence this article!

自从发布这些文章，有很多人要求我是否能阐述下更多复杂的 RAC3 代码，这也是我写这文章的原因。

## A QUICK MVVM REFRESHER

## 一个快速的 MVVM 刷新机制

ReactiveCocoa is not an MVVM framework in itself, however, it provides functionality that make it easier to implement apps using this popular UI pattern.

ReactiveCocoa 本身并不是一个 MVVM 的框架，然而，它让那些使用流行的 UI 模式制作 app 变得更加简单。

At the core of this pattern is the ViewModel which is a special type of model that represents the UI state of the application. It contains properties that detail the state of each and every UI control, for example the current text for a text field or whether a specific button is enabled. It also exposes the actions that the view is able to perform, for example button taps or gestures.

这种模式的核心是 ViewModel。它是一个特殊类型的 model，反映 app 的 UI 状态。它包含每个 UI 的详细状态，比如当前 textField 的文本内容，或者按钮的 enable 状态。它也暴露了视图的响应操作，比如按钮点击、手势。

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMPattern.png)

Looking at the MVVM pattern specifically from the perspective of iOS development, the View is composed of the ViewController plus its associated UI (whether that is a nib, storyboard or constructed though code):

从 iOS 开发层面来具体看 MVVM 模式，视图就是控制器用来显示 UI 的组件（无论是 nib，storyboard 还是代码生成的）

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMReactiveCocoa.png)

With MVVM your Views should be very simple, doing little more than just reflecting the current UI state.

使用 MVVM 会使得视图变得简单，只需要做很少的处理来反映当前 UI 的状态。

ReactiveCocoa holds a special role in implementing MVVM applications, providing a simple mechanism for synchronising Views and their associated ViewModels.

ReactiveCocoa 在 MVVM 应用中扮演者特殊的角色，提供很简单的机制来同步视图和它所关联的 ViewModel。

## MVVM IN REACTIVECOCOA 2.0

With RAC2, the process of binding ViewModel properties to your View involved the use of a number of macros:

在 RAC2，将 ViewModel 上的属性绑定到视图上通常需要使用一堆的宏：

``` 
RAC(self.loadingIndicator, hidden) = self.viewModel.executeSearch.executing;
```

The above code binds the `hidden` property of a loading indicator to the `executing` signal on the view model via the `RAC` macro. Another useful RAC2 macro is `RACObserve` which creates a signal from a property, effectively acting as a wrapper around KVO.

上述代码通过宏将 loadingIndicator 的 `hidden` 的属性绑定到 ViewModel 的 `executing` signal 上。另一个有用的宏是 `RACObserve`，可以从属性中生成一个信号，由于是基于 KVO，执行效率很高。

(If you’ve not used MVVM with ReactiveCocoa before, you might want to read my [tutorial on Ray Wenderlich’s site](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1))

（如果你之前没有在 MVVM 中使用 ReactiveCocoa，你可以看看我之前的文章 [tutorial on Ray Wenderlich’s site](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)）

Unfortunately these RAC2 macros are pretty clumsy and cumbersome to use. This is true of all macro-based APIs, not an issue with RAC specifically.

不幸的是，RAC2 这些宏使用起来有些笨拙和累赘。所有基于宏的 API 都是如此，不单单只是 RAC 自己。

RAC3 does away with macros, and KVO, replacing them both with a pure Swift implementation.

RAC3 不再使用宏和 KVO，而是单纯 Swift 风格的实现。

## RAC3 PROPERTIES

## RAC3 属性

I wrote a blog post a few months back which looked at [KVO and a few KVO-alternatives with Swift](http://blog.scottlogic.com/2015/02/11/swift-kvo-alternatives.html), the lack of strong-typing, dependence on NSObject and a rather clumsy syntax mean that KVO feels quite uncomfortable within the swift world.



几个月前，我写了一篇 [KVO and a few KVO-alternatives with Swift](http://blog.scottlogic.com/2015/02/11/swift-kvo-alternatives.html)，主要说的是，由于缺少强类型，过度依赖 NSObject 和过于笨拙的语法，意味着 KVO 并不再适合 swift 的世界。



With RAC3, properties (or at least properties which you wish to observe),are represented by the generic `MutableProperty` type:



在 RAC3，属性（至少是你要监听的属性）主要使用 `MutableProperty` 泛型：

``` 
let name = MutableProperty<String>("")
```

This creates a `name` property of type `String` and initialises it with an empty string. Notice that the name property above is a constant, despite the fact that it represents a mutable property!

这个实例化了一个 `name` 属性，类型是 `String`，初始值是空字符串。注意 name 这个属性是个常量（用的是 let），尽管它代表着一个可变的属性。

Mutable properties have a very simple API, with a `value` property and `put` method, allowing you to get / set the current value:

可变属性有一个很简单的 API，一个 `value` 属性和一个 `put` 方法，允许你 get 或 set 操作当前的值：

``` 
name.put("Frank")
println(name.value)
```

They also expose a `producer` property of type `SignalProducer`, which allows you to observe property changes:

他们还暴露了一个类型为`SignalProducer` 的 `producer` 属性，允许你观察属性的变化：

``` 
name.producer
  |> start(next: {
    println("name has changed to value \($0)")
  })
```

… with everything being all nice and strongly typed.

With the MVVM pattern the process of the ViewModel to the View typically involves binding properties together, in other words, you want to ensure that the various properties of your view are synchronised with the respective ViewModel properties.

RAC3 has a specific operator for this purpose:

...每个都很清晰，强类型。

在 MVVM 模式，配置一个 ViewModel 通常都会绑定一个属性。换句话说，就是确保视图的变量属性与对应的 ViewModel 属性保持同步。

出于这个目的，RAC3 有个明确的操作符：

``` 
executionTimeTextField.rac_text  <~ viewModel.queryExecutionTime
```

The above code binds the `queryExecutionTime` view model property to the `rac_text` property on the text field.

上述代码，viewModel 的 `queryExecutionTime` 绑定了 textField 的 `rac_text` 属性。

**NOTE:** Currently [RAC3 does not support two-way binding](https://github.com/ReactiveCocoa/ReactiveCocoa/issues/1986).

**注意：** 当前 [RAC3 不支持 2 种方式的绑定](https://github.com/ReactiveCocoa/ReactiveCocoa/issues/1986)。

## AN MVVM RAC3 EXAMPLE

## 一个 MVVM RAC3 的例子

As promised, this blog post includes a more in-depth RAC3 example. A twitter search example:

作为约定，这篇博客包括了一个更深入的 RAC3 的例子。一个 twitter 的搜索 demo：

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMRAC3.png)

(Yes, all my example code seems to involve either Twitter or Flickr APIs! - if you have some more creative ideas that you’d like to share, please do)

（对的，我的所有代码都与 Twitter 和 Flicker API 有关，如果你在天朝，自重）

The app searches for tweets containing the given text, automatically executing the search as the user types.

这个 app 会根据给定的 text 来搜索 tweet，自动根据用户类型执行搜索。

The ViewModel that backs the application has the following properties:

TwitterSearchViewModel 有如下这些属性： 

``` 
class TwitterSearchViewModel {

  let searchText = MutableProperty<String>("")
  let queryExecutionTime = MutableProperty<String>("")
  let isSearching = MutableProperty<Bool>(false)
  let tweets = MutableProperty<[TweetViewModel]>([TweetViewModel]())

  private let searchService: TwitterSearchService

  ...
}
```

These represent everything the View needs to know about the current UI state, and allow it to be notified, via RAC3 bindings, of updates. The table view of tweets is ‘backed’ by the `tweets` mtable property which contains an array of ViewModel instances, each one backing an inidividual cell.

所有这些属性是用来反映视图上的 UI 状态，并且允许通过 RAC3 绑定被响应更新。tweet 的 tableView 通过 `tweets` 可变属性被‘返回’，它包含一个数组，里面是 ViewModel 实例，每个都是对应独立的 cell。

The `TwitterSearchService` class provides a RAC3 wrapper around the Twitter APIs, representing requests as signal producers.

`TwitterSearchService` 类是一个基于 RAC3，封装了 Twitter API 的封装，代表了网络请求，类型是 signal producer。

The core pipeline for this application is as follows:

这个应用的核心代码如下：

``` 
let textToSearchSignal: SignalProducer<String, NSError>  -> SignalProducer<TwitterResponse, NSError> =
  flatMap(.Latest) {
    text in self.searchService.signalForSearchWithText(text)
  }

searchService.requestAccessToTwitterSignal()
  |> then(searchText.producer |> mapError { _ in TwitterInstantError.NoError.toError() })
  |> filter {
      count($0) > 3
    }
  |> throttle(1.0, onScheduler: QueueScheduler.mainQueueScheduler)
  |> on(next: {
      _ in self.isSearching.put(true)
    })
  |> textToSearchSignal
  |> observeOn(QueueScheduler.mainQueueScheduler)
  |> start(next: {
      response in
      self.isSearching.put(false)
      self.queryExecutionTime.put("Execution time: \(response.responseTime)")
      self.tweets.put(response.tweets.map { TweetViewModel(tweet: $0) })
    }, error: {
      println("Error \($0)")
    })
```

This requests access to the user’s twitter account, following this the pipeline passes control to the`searchText.producer`, i.e. it observes its own `searchText` property. You’ll notice that the producer isn’;’t used directly, instead it is first mapped as follows: `searchText.producer |> mapError`. This highlights a common issue with RAC3, because signals have an error type constraint, any operation that combines signals (or signal producers) requires that their error types matches. The use of`mapError` above transforms any error that `searchText.producer` might produce into an `NSError`, which is compatible with the other signals being used in this pipeline.

这个请求访问了用户 twitter 账户，沿着这个管道将操作传给 `searchText.producer`，例如，它观察了它自己的 `searchText` 属性。你将会注意到 producer 后面并没有直接结束，而是先调用 map 方法 `searchText.producer |> mapError`。这个在 RAC3 引起了一些讨论，因为 signal 有个 error 类型的常量，任何操作合并了 signal （或者是 signal producers）就必须要求他们之间的 error 类型是相匹配的。使用 `mapError` 转化任意错误，即 `searchText.producer` 可以生成一个 `NSError`，可以在这个管道和其他 signal 相匹配。

Following this, the signal is filtered and throttled. This reduces the frequency of the signal if the`searchText` property (which is bound to the UI), changes rapidly.

再往下，signal 被过滤（filter）和节流(throttle)。如果 `searchText` 多次变动，这可以减少 signal 的产生。

The signal is then flat-mapped to a signal that searches twitter based on the given search text. Notice that I had to ‘break out’ the `flatMap` operation. This is due to the overloaded nature of this operation making it impossible for the compiler to determine which implementation to use - I’ll raise an issue on GitHub for that one shortly!

然后 signal 被 flat-mapped 到另一个 signal，根据给定的 text 搜索 twitter。注意我已经打破了 `flatMap` 操作。这个是由于重载了原生的操作符，使得编译器无法决定该执行那个。我已经在 Github 提了一个 issue，相信很快会解决。

## ADDING MUTABLE PROPERTIES TO UIKIT

Currently RAC3 lacks any UIKit integration, my guess is that this will come later in the beta process. For now, in order to bind a ViewModel to a UIKit View, you have to add the required extensions yourself. Fortunately this is quite simple!

In another side project I have been messing about with I created a utility function for creating lazily-constructed associated properties:

当前 RAC 3 缺少一些 UIKit 集成，我猜想它会在下个 beta 版出现。现在，为了给视图绑定一个 ViewModel，你不得不自己添加 extension。幸运的是这很简单。

在另外一个项目，我已经创建了一个有用的函数，用来懒加载关联属性：



``` 
func lazyAssociatedProperty<T: AnyObject>(host: AnyObject,
                       key: UnsafePointer<Void>, factory: ()->T) -> T {
  var associatedProperty = objc_getAssociatedObject(host, key) as? T

  if associatedProperty == nil {
    associatedProperty = factory()
    objc_setAssociatedObject(host, key, associatedProperty,
                                   UInt(OBJC_ASSOCIATION_RETAIN))
  }
  return associatedProperty!
}
```

This creates a property of type `T`, where the given factory function is used to create the properties initial value on first access.

This can be used to create a lazily-constructed mutable property:

这里创建了一个 `T` 类型的属性，给定的比例函数式用于第一次访问初始化属性。

这个可以被用于懒加载一个可变属性：



``` 
func lazyMutableProperty<T>(host: AnyObject, key: UnsafePointer<Void>,
              setter: T -> (), getter: () -> T) -> MutableProperty<T> {
  return lazyAssociatedProperty(host, key) {
    var property = MutableProperty<T>(getter())
    property.producer
      .start(next: {
        newValue in
        setter(newValue)
      })
    return property
  }
}
```

The above code creates a mutable property, then subscribes to the producer, calling the supplied`setter` function when the value changes.

This can be used to add RAC3 properties to a view as follows:

上面的代码创建了一个可变的属性，然后想 producer 订阅，当值改变的时候调用 `setter` 函数。

这个可以被用于添加 RAC3 属性到视图上，如下：



``` 
extension UIView {
  public var rac_alpha: MutableProperty<CGFloat> {
    return lazyMutableProperty(self, &AssociationKey.alpha, { self.alpha = $0 }, { self.alpha  })
  }

  public var rac_hidden: MutableProperty<Bool> {
    return lazyMutableProperty(self, &AssociationKey.hidden, { self.hidden = $0 }, { self.hidden  })
  }
}
```

Note that these properties only have getters, you set them via the `put` method on the property itself.

Within this project I have only added the properties I need for this example code.



注意，这些属性只有 getter 方法，你得通过 `put` 方法设置。

在这个项目我只添加我需要的属性：



It is of course a little more complicated for controls where they also mutate the same properties. With a text field you also have to subscribe to changes as a result of user input:

对于控件来说，创建这些相同的属性有些复杂。在一个 textField 你必须订阅信号变化，作为用户输入结果：



``` 
extension UITextField {
  public var rac_text: MutableProperty<String> {
    return lazyAssociatedProperty(self, &AssociationKey.text) {

      self.addTarget(self, action: "changed", forControlEvents: UIControlEvents.EditingChanged)

      var property = MutableProperty<String>(self.text ?? "")
      property.producer
        .start(next: {
          newValue in
          self.text = newValue
        })
      return property
    }
  }

  func changed() {
    rac_text.value = self.text
  }
}
```

With this in place, the ViewModel can be bound as follows:

这里，ViewModel 可以被绑定：



``` 
viewModel.searchText <~ searchTextField.rac_text
executionTimeTextField.rac_text  <~ viewModel.queryExecutionTime
```

It’s also possible to map values as part of the binding process.

这里还可以对绑定的值做 map 操作：



``` 
searchAcitivyIndicator.rac_hidden <~ viewModel.isSearching.producer
                                       |> map { !$0 }
tweetsTable.rac_alpha <~ viewModel.isSearching.producer
                            |> map { $0 ? CGFloat(0.5) : CGFloat(1.0) }
```

For the tweets table view, this project also includes an adapted version of the [table view binding code](http://blog.scottlogic.com/2014/05/11/reactivecocoa-tableview-binding.html) I wrote a while back.

Interestingly, RAC3 also has a `ConstantProperty` class, which might seem a little odd! I use it for the ViewModel that backs each table cell:

对于 tableView，这个项目还包括了 [tableView 绑定代码](http://blog.scottlogic.com/2014/05/11/reactivecocoa-tableview-binding.html)的版本。

有趣的是，RAC3 还有个 `ConstantProperty` 类，看起来有点奇怪！我把它用在了 ViewModel，用来返回每个 cell：



``` 
class TweetViewModel: NSObject {

  let status: ConstantProperty<String>
  let username: ConstantProperty<String>
  let profileImageUrl: ConstantProperty<String>

  ...
}
```

The value of this `ConstantProperty` is that you can still use the `<~` binding operator, and in future if you do decide to make it mutable you do not have to change your binding code.

`ConstantProperty` 的值也是可以使用 `<~` 绑定操作，而且在未来，如果你决定使它成为可变，你不需要在改变代码。

## CONCLUSIONS

## 总结

RAC3 is shaping up to be a really great framework. There are still one or two loose ends, but overall it represents is a significant step forwards.

RAC3 是一个快速发展的强大框架。这里也聊到一两点，但是也不难看出它代表了一个里程碑。

All the code for this example app is [available on GitHub](https://github.com/ColinEberhardt/ReactiveTwitterSearch). I’d also suggest taking a look at[WhiskyNotebook](https://github.com/nebhale/WhiskyNotebook), another project which makes quite a bit of ue fo RAC3.

所有 demo 的代码都可以在 [Github](https://github.com/ColinEberhardt/ReactiveTwitterSearch) 上找到。建议也看下另外一个 [WhiskyNotebook](https://github.com/nebhale/WhiskyNotebook) 项目，它也用到了 RAC3.