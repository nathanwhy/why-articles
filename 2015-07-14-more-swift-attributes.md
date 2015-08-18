# MORE SWIFT ATTRIBUTES

# 一些少见的 Swift 属性

Swift has a variety of little documented (or undocumented) attributes just sitting there waiting to be used. Let's look at a few of them:

Swift 有各种在苹果文档中鲜有记载或未记载的属性。他们正等待着你们去使用。让我们看看有哪些：

###`@INLINE`

This attribute gives the compiler inlining hints. The valid values are __always and never. I don't think I'd use this one (especially __always) unless I was absolutely certain I needed it; the rules around it are not currently known. In limited testing it seems to work but YMMV.

这个属性提供编译器内联提示。有效的值为 `__always` 和 `never`。除非我非常确定需要，否者我不会使用这个（特别是 `__always`）。关于它的使用方式还不是很清楚。在一些有限的测试中它还能生效，但不同测试环境效果也可能不同。

edit: To further explain: though LLVM has the concept of forced inlining, we don't currently know if this attribute actually maps to that directly. Nor do we know if there are size limits that will cause the compiler to ignore this and skip inlining. In theory it should have this behavior but I'm not going to promise anyone that it does.

修改：进一步解释，尽管 LLVM 有强制内联的概念，但我们现在并不清楚这个属性是否与其直接关联。我们也不清楚是否有大小限制导致编译器忽略它，跳过内联。理论上它本应该有这个行为，但是我不敢保证它有。

Note that @inline attributes are ignored in debug builds (when optimizations are turned off).

记住 `@inline` 属性在 debug 编译下会被忽略（当 optimization 被关闭）。

(译者注：关于 optimization 可以看[这篇文章](http://ios.jobbole.com/81937/))

###`@TRANSPARENT`

I originally left this one out of the list. It causes the compiler to inline the function much earlier in the pipeline. It is intended for "very primitive functions like +(Int, Int)" that should "never be emitted as independent functions".

我原本不把这个列入清单。它会使编译器更早地在构建流程中进行内联函数。它的作用是使["像(Int, Int)这种非常简单的函数"不应该是一个独立的函数"](https://devforums.apple.com/message/988972#988972)

`@transparent` functions are inlined even in debug mode with no optimizations, so things like `1 + 1` aren't horribly slow function calls. Otherwise it works very similarly to `@inline(__always)`.

`@transparent` 函数是内联的，即使是在没有 optimization 的 debug 模式下, 所以像 `1 + 1` 这种简单函数也可以调用运行很快. 否则它的作用就像是 `@inline(__always)`.

###`@AVAILABILITY`


This attribute marks things as only available on certain platforms or in certain versions. The first parameter is the platform. Can be `*` for all, `iOS`, or `OSX`. You can specify multiple `@availability` attributes if necessary for different platforms.

这个属性标记那些只在某些特定版本或平台上有效的对象。第一个参数是平台。可以是 `*`（所有）、`iOS` 或 `OSX`。如果需要针对多个不同平台，可以指定多个 `@availability` 属性。

The second parameter can be `unavailable` which indicates that this item is not available at all on the given platform. Otherwise you can specify a combination of one or more versions: `introduced`, `deprecated`, and `obsoleted`. Obsoleted means the item was removed, deprecated just means it will give a warning if used. Lastly you can specify `message` which will be output by the compiler if the item is used. Some examples:

第二个参数可以是 `unavailable`，表明对于给定的平台不可用。相对地，你可以声明一个或多个版本的组合:`introduced`, `deprecated`, 和 `obsoleted`。Obsoleted 意味着已被移除，deprecated 表示如果使用就会给出警告。最后一个参数你可以添加 `message`，当被使用的时候，编译器会输出这些提示。一些例子：

```
@availability(*, unavailable)
func foo() {}

@availability(iOS, unavailable, message="you can't call this")
func foo2() {}

@availability(OSX, introduced=10.4, deprecated=10.6, obsoleted=10.10)
@availability(iOS, introduced=5.0, deprecated=7.0)
func foo3() {}
```

###`@NORETURN`

Just like it says: the compiler can assume this function is either the beginning of an eternal run loop, `while true { }`, or it aborts or exits the process.

如同名字的意思一样，编译器会假定这个函数是一个永恒 run loop 的开始，`while true { }`，或者这个函数终结、退出当前进程。

Edit: Commenter Marco Masser points out that the compiler will ignore missing return values in a function if you call another function marked `@noreturn` because it understands the control flow.

修改：有评论说如果你调用另一个被 `@noreturn` 标记的函数，这个编译器会无视一个忘记返回值的函数，因为它了解控制流。

###`@ASMNAME`

Gives the symbol name for the function, method, or property's implementation. If you can figure out the parameters and their types then this will let you call internal Swift standard library functions... or even C functions for which you don't have the headers handy: @asmname("function") func f()

给函数、方法、或属性的实现一个标记名字。如果你找到了参数和他们的类型说明，使用这个标记你可以调用 Swift 内部标准库的函数...或者甚至是没有头文件的 C 函数，`@asmname("function") func f()`。

###`@UNSAFE_NO_OBJC_TAGGED_POINTER`

This one is still a mystery but my guess is it tells Swift not to use tagged pointers when bridging to Objective-C.

这个标记仍然是一个迷，但我猜是用来告诉编译器，当桥接 Objective-C 时不要使用 tagged pointers。

(译者注：[`tagged pointers`](http://blog.devtang.com/blog/2014/03/21/weak_object_lifecycle_and_tagged_pointer/)。)

###`@SEMANTICS`

Yet another mystery. The parameters seem to be strings like array.mutate_unknown or array.init. Presumably this gives the compiler (or the static analyzer) information about how the function behaves.

另外一个谜。参数似乎是 string 类型，类似 `array.mutate_unknown` 或 `array.init`。这大概会告诉编译器（或静态分析器）一些函数行为的信息。

##CONCLUSION
##总结

Who needs boring old @objc or @autoclosure? Get out there and live a little.

是否已经玩腻了 `@objc` 和 `@autoclosure`，试试这些。

