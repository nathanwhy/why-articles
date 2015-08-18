这个是我最后一篇关于 ReactiveCocoa 3.0 （简称 RAC3）的文章，主要介绍了在 MVVM 实践中更多复杂的 RAC 3.0 的用法。

(译者注：前两篇为 [ReactiveCocoa 3.0 初见](http://ios.jobbole.com/82081/)和 [ReactiveCocoa 3.0 初见(2)](http://ios.jobbole.com/82125/))

ReactiveCocoa 3.0 当前还处于测试阶段，截止本文发表当天，已经有 [4 个 beta 版](https://github.com/ReactiveCocoa/ReactiveCocoa/releases)。相较于 Objective-C 版本，3.0 版本带来了全新的 Swift API。尽管函数响应式编程的核心理念保持不变，但是 Swfit API 相比于之前的版本却有很大的不同。它使用了泛型，自定义操作符和柯里化函数，实现效果相当不错。在我前面的几个版本中，我探讨了一般性的 [Signal 类](http://blog.scottlogic.com/2015/04/24/first-look-reactive-cocoa-3.html)，它提供了强类型的信号管道模型，还有一篇关于 [SignalProducer 的类接口](http://blog.scottlogic.com/2015/04/28/reactive-cocoa-3-continued.html)，SignalProducer 为具有附带作用的信号提供了一种更清晰的表达方式。

自从发布这些文章，有很多人请求我演示更多复杂的 RAC3 代码例子，于是我就写下这篇文章。

## 一个快速的 MVVM 刷新机制

ReactiveCocoa 本身并不是一个 MVVM 的框架，然而，它让那些使用 MVVM 这种流行的 UI 架构的 app 变得更加容易构建。

这种模式的核心是 ViewModel。它是一个特殊类型的 model，反映 app 的 UI 状态。它包含每个 UI 的详细状态，比如当前 textField 的文本内容，或者按钮的 enable 状态。它也提供了视图可以进行的响应操作，比如响应按钮的点击或者手势识别的响应操作。

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMPattern.png)

从 iOS 开发层面来具体看 MVVM 模式，View 由 ViewController 和对应的 UI 组成（无论是 nib，storyboard 还是代码生成）

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMReactiveCocoa.png)

使用 MVVM 会使得 View 层变得简单，只需进行反映当前的 UI 状态和其他一点点工作。

ReactiveCocoa 在 MVVM 应用中扮演者特殊的角色，提供一个简单的途径来同步视图和它所关联的 ViewModel。

## 在 REACTIVECOCOA 2.0 中进行 MVVM

在 RAC2，将 ViewModel 上的属性绑定到视图上通常需要使用一堆宏：

``` 
RAC(self.loadingIndicator, hidden) = self.viewModel.executeSearch.executing;
```

上述代码通过宏将 loadingIndicator 的 `hidden` 的属性绑定到 ViewModel 的 `executing` signal 上。另一个有用的宏是 `RACObserve`，可以从属性中生成一个信号，作用相当于一个对 KVO 的高效封装。

（如果你之前没有在 MVVM 中使用 ReactiveCocoa，你可以看看我之前的文章 [tutorial on Ray Wenderlich’s site](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)）

不幸的是，RAC2 这些宏使用起来有些笨拙和累赘。所有基于宏的 API 都是如此，不单单只是 RAC 这些。

RAC3 不再使用宏和 KVO，而是单纯 Swift 风格的实现。

## RAC3 属性

几个月前，我写了一篇 [KVO and a few KVO-alternatives with Swift](http://blog.scottlogic.com/2015/02/11/swift-kvo-alternatives.html)，主要说的是缺少强类型，过度依赖 NSObject 和过于笨拙的语法，意味着 KVO 并不适合 swift 的世界。

在 RAC3，属性（至少你要监听的属性）使用 `MutableProperty` 类型表示：

``` 
let name = MutableProperty<String>("")
```

这个实例化了一个 `name` 属性，类型是 `String`，初始值是空字符串。注意 name 这个属性是个常量（用的是 let），尽管它代表着一个可变的属性。

可变属性有一个很简单的 API，一个 `value` 属性和一个 `put` 方法，允许你 get 或 set 操作当前的值：

``` 
name.put("Frank")
println(name.value)
```

他们还暴露了一个类型为`SignalProducer` 的 `producer` 属性，允许你观察属性的变化：

``` 
name.producer
  |> start(next: {
    println("name has changed to value \($0)")
  })
```

...每一步都很清晰，并且是强类型。

在 MVVM 模式，配置一个 ViewModel 通常都会绑定一个属性。换句话说，就是确保视图的变量属性与对应的 ViewModel 属性保持同步。

出于这个目的，RAC3 有个明确的操作符：

``` 
executionTimeTextField.rac_text  <~ viewModel.queryExecutionTime
```

上述代码，viewModel 的 `queryExecutionTime` 绑定了 textField 的 `rac_text` 属性。

**注意：** 当前 [RAC3 不支持双向绑定](https://github.com/ReactiveCocoa/ReactiveCocoa/issues/1986)。

## 一个 MVVM RAC3 的例子

作为约定，这篇博客包括了一个更深入的 RAC3 的例子。一个 twitter 的搜索 demo：

![](http://blog.scottlogic.com/ceberhardt/assets/rac3/MVVMRAC3.png)

（对的，我的所有代码都与 Twitter 和 Flicker API 有关，如果你在天朝，自重）

这个 app 会根据给定的 text 来搜索 tweet，搜索会在用户输入后自动执行。

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

这些属性就是所有 View 反映视图 UI 状态需要知道的属性，并且允许通过 RAC3 绑定响应更新。tweet 的 tableView 通过 `tweets` 可变属性被‘返回’，它包含一个数组，里面是 ViewModel 实例，每个都对应一个独立的 cell。

`TwitterSearchService` 类是一个基于 RAC3 Twitter API 的封装，用 signal producers 代表了网络请求。

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

这个请求访问了用户 twitter 账户，接着管道将操作传给 `searchText.producer`，也就是它观察了它自己的 `searchText` 属性。你将会注意到 producer 并没有直接使用，而是先调用 map 方法 `searchText.producer |> mapError`。这在 RAC3 中是一个常见问题，因为 signal 有个 error 类型的常量，任何操作合并 signal （或者是 signal producers）就必须要求他们之间的 error 类型是相匹配的。使用 `mapError` 转化 `searchText.producer` 可能产生的任意错误为一个 `NSError`，与管道中其他 signal 相匹配。

再往下，signal 被过滤（filter）和节流(throttle)。如果 `searchText` 多次变动，这可以减少 signal 的产生。

然后 signal 被 flat-mapped 到另一个 signal，根据给定的 text 搜索 twitter。注意我已经打破了 `flatMap` 操作。这个是由于重载了原生的操作符，使得编译器无法决定该执行那个。马上我会在 Github 提一个 issue。

## 为 UIKit 添加可变属性

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

上面的代码创建了一个可变的属性，然后向 producer 订阅，当值改变的时候调用 `setter` 函数。

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

注意，这些属性只有 getter 方法，你得通过属性本身的 `put` 方法为他们赋值。

在这个项目我只添加了我需要的属性：

对于控件来说，如果他们也会改变同样的属性，这种情况就自然会更复杂。在一个 textField 你必须订阅用户输入结果的变化：

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

这里，ViewModel 可以被绑定：

``` 
viewModel.searchText <~ searchTextField.rac_text
executionTimeTextField.rac_text  <~ viewModel.queryExecutionTime
```

这里还可以对绑定的值做 map 操作：

``` 
searchAcitivyIndicator.rac_hidden <~ viewModel.isSearching.producer
                                       |> map { !$0 }
tweetsTable.rac_alpha <~ viewModel.isSearching.producer
                            |> map { $0 ? CGFloat(0.5) : CGFloat(1.0) }
```

对于 tableView，这个项目还包括了 [tableView 绑定代码](http://blog.scottlogic.com/2014/05/11/reactivecocoa-tableview-binding.html)的版本。

有趣的是，RAC3 还有个 `ConstantProperty` 类，看起来有点奇怪！我把它用在了构建每个 cell 的 ViewModel：


``` 
class TweetViewModel: NSObject {

  let status: ConstantProperty<String>
  let username: ConstantProperty<String>
  let profileImageUrl: ConstantProperty<String>

  ...
}
```

`ConstantProperty` 的值也是可以使用 `<~` 绑定操作，而且在未来，如果你决定使它成为可变，你不需要在改变代码。

## 总结

RAC3 是一个快速发展的强大框架。也有一两点缺陷，但是也不难看出它代表了一个里程碑。

所有 demo 的代码都可以在 [Github](https://github.com/ColinEberhardt/ReactiveTwitterSearch) 上找到。建议也看下另外一个 [WhiskyNotebook](https://github.com/nebhale/WhiskyNotebook) 项目，它也用到了 RAC3.