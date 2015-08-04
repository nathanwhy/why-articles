## Swift Functors, Applicatives 和 Monads 的图文解释

> This is a translation of [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) from [Haskell](https://www.haskell.org/) into Swift.
> 
> I don't want to take any merit for writing this, I only went through the fun exercise of translating the code snippets in Swift.
> 
> If you enjoy this post be sure to say thanks to the author of the original version: [Aditya Bhargava](http://adit.io/index.html),[@_egonschiele](https://twitter.com/_egonschiele) on Twitter.
> 
> Despite all the hype about it, **Swift is not a functional language**. This means that we need to write a bit of extra code to achieve the same results that language like Haskell have with built-in operators.
> 
> You can find a Playground with all the code from the article [on GitHub](https://github.com/mokacoding/Swift-Functors-Applicative-Monads-In-Pictures-Playground).
> 
> Finally, don't worry if you find the content hard to grasp. I had to read the original version a number of times to wrap my head around it, plus a lot of mess around with the Swift code.
> 
> 这是一篇 [Haskell 官网](https://www.haskell.org/) 文章 [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) 的 Swift 移植版本。
> 
> 创作这篇文章的并不是我，我只是将它其中的代码翻译为 Swift ，这个过程很有趣。
> 
> 如果你喜欢这篇文章，请感谢原作者 [Aditya Bhargava](http://adit.io/index.html),[@_egonschiele](https://twitter.com/_egonschiele)
> 
> 尽管关于 Swift 的宣传沸沸扬扬，但是它并不是一门真正的函数式语言。这意味着要实现一些 Haskell 内置的操作符功能，我们可能需要写一些额外的代码。
> 
> 你可以在 [GitHub](https://github.com/mokacoding/Swift-Functors-Applicative-Monads-In-Pictures-Playground) 找到包含本文所有代码的 playground。
> 
> 最后，如果你觉得这篇文章内容对于你来说很难理解不要担心，我也是在将原文读了许多遍之后才渐渐找到思路，而在翻译为 Swift 的过程中依然是一团糟。

Here’s a simple value:

这是一个简单的值（value）：

![](http://adit.io/imgs/functors/value.png)

And we know how to apply a function to this value:

我们也知道如何使用函数（function）来处理值：

![](http://adit.io/imgs/functors/value_apply.png)

Simple enough. Lets extend this by saying that any value can be in a context. For now you can think of a context as a box that you can put a value in:


这很容易懂，那么拓展一下，任意值都能在处于特定的上下文（context）中。你可以先想象上下文就像是一个盒子，你可以把值放进去。

![](http://adit.io/imgs/functors/value_and_context.png)

Now when you apply a function to this value, you’ll get different results **depending on the context**. This is the idea that Functors, Applicatives, Monads, Arrows etc are all based on. The`Optional` type defines two related contexts:


现在当你使用函数处理这个值，根据上下文的不同会得到不同的结果。Functors, Applicatives, Monads，Arrows等概念都是基于此。`Optional` 类型定义了两种相关的上下文:

> **Note:** the pictures use Maybe (Just | None) from Haskell, which correspond to Swift's Optional .Some and .None.
> 
> 注意：图上的 Maybe（Just | None）来自 Haskell，类似于 Swift 的 Optional，`.Some` 和 `.None`。

![](http://adit.io/imgs/functors/context.png)

``` 
enum Optional<T> {
  case None
  case Some(T)
}
```

In a second we will see how function application is different when something is a `.Some(T)`versus a `.None`. First let’s talk about Functors!


紧接着我们将看到一个值的类型是 `.Some(T)` 或者是 `.None` 会怎样造成函数作用的不同。我们先来谈谈 Functors！



## Functors

## Functors

When a value is wrapped in a context, you can’t apply a normal function to it:


当一个值被封装到盒子里，一个普通的函数无法作用于它：

![](http://adit.io/imgs/functors/no_fmap_ouch.png)

This is where `map` comes in (`fmap` in Haskell). `map` is from the street, `map` is hip to contexts.`map` knows how to apply functions to values that are wrapped in a context. For example, suppose you want to apply a function that adds 3 to `.Some(2)`. Use `map`:


这个是就是 `map` 的由来（在 Haskell 是 `fmap`）。`map` 知道如何使用函数处理数据类型。例如，你想要使用一个函数，将 `.Some(2)` 加 3。使用 `map`:

``` 
func plusThree(addend: Int) -> Int {
  return addend + 3
}

Optional.Some(2).map(plusThree)
// => .Some(5)

```

or with a simple syntax using Swift's autoclosure:


或者用更简洁的语法，使用 Swift 的 autoclosure：

``` 
Optional.Some(2).map { $0 + 3 }
// => .Some(5)
```

![](http://adit.io/imgs/functors/fmap_apply.png)

**Bam!** `map` shows us how it’s done! But how does `map` know how to apply the function?


**砰!** `map` 的作用我们看到了，但是它是怎么做到的？

## Just what is a Functor, really?

## 到底什么是 Functor？

A Functor is any type that defines how `map` (`fmap` in Haskell) applies to it. Here’s how `map`works:


任意定义了 `map` （ Haskell 中的 `fmap`）如何作用于自己的类型都是 Functor，`map`是这样作用的：

![](http://adit.io/imgs/functors/fmap_def.png)

So we can do this:

所以我们可以这么做

``` 
Optional.Some(2).map { $0 + 3 }
// => .Some(5)

```

And `map` magically applies this function, because `Optional` is a Functor. It specifies how `map` applies to `Some`s and `None`s:


`map` 神奇地使函数起了作用，因为 `Optional` 是一个 Functor。它表明了 `map` 是如何应用 `Some` 和 `None`。

``` 
func map<U>(f: T -> U) -> U? {
  switch self {
  case .Some(let x): return f(x)
  case .None: return .None
}

```

Here’s what is happening behind the scenes when we write `Optional.Some(2).map { $0 + 3 }`:


这里是当我们写下 `Optional.Some(2).map { $0 + 3 }` 背后所发生的：

![](http://adit.io/imgs/functors/fmap_just.png)

So then you’re like, alright `map`, please apply `{ $0 + 3 }` to a `.None`?

所以我们就像在说，`map`，请将 `{ $0 + 3 }`作用与 `.None` 上？

![](http://adit.io/imgs/functors/fmap_nothing.png)

``` 
Optional.None.map { $0 + 3 }
// => .None

```

![](http://adit.io/imgs/functors/bill.png)

*Bill O’Reilly being totally ignorant about the Maybe functor*

Like Morpheus in the Matrix, `map` knows just what to do; you start with `None`, and you end up with `None`! `map` is zen. Now it makes sense why the `Optional` type exists. For example, here’s how you work with a database record in a language without `Optional`, like Ruby:


就像黑客帝国中的 Morpheus，`map` 知道要做什么；开始时是 `None`，结束也是 `None`！`map` 是一种禅。现在你可以理解为什么 `Optional` 类型的存在。举个例子，对于没有 `Optional` 类型的语言，比如 Ruby，对于一条数据库记录是这么工作的：

``` 
let post = Post.findByID(1)
if post != nil {
  return post.title
} else {
  return nil
}

```

But in with Swift using the `Optional` functor:


但是用 Swift 使用 `Optional` 仿函数：

``` 
findPost(1).map(getPostTitle)

```

If `findPost(1)` returns a post, we will get the title with `getPostTitle`. If it returns `None`, we will return `None`!


如果 `findPost(1)` 返回 post，我们会通过 `getPostTitle` 得到 title。如果他返回 `None`，我们会返回 `None`!


We can even define an infix operator for `map`, `<^>` (`<$>` in Haskell), and do this instead:


我们甚至可以定义一个 infix 操作符给 `map`,`<^>` (在 Haskell 为`<$>`），然后这么做：

``` 
infix operator <^> { associativity left }

func <^><T, U>(f: T -> U, a: T?) -> U? {
  return a.map(f)
}

getPostTitle <^> findPost(1)

```

> **Note:** we have to use `<^>` because `<$>` wouldn't compile.



> **注意：**我们使用`<^>`，因为`<$>`不能编译通过。

Here’s another example: what happens when you apply a function to an array?



这里有另一个例子：对 array 使用函数会怎么样呢？

![](http://adit.io/imgs/functors/fmap_list.png)

Arrays are functors too!

Array 也是 functor！

Okay, okay, one last example: what happens when you apply a function to another function?

好了，最后一个例子：给函数应用另外一个函数会怎么样？

``` 
map({ $0 + 2 }, { $0 + 3 })
// => ???

```

Here's a function:


这个是函数：

![](http://adit.io/imgs/functors/function_with_value.png)

Here’s a function applied to another function:


这个是一个函数应用另外一个函数：

![](http://adit.io/imgs/functors/fmap_function.png)

The result is just another function!


获得结果是另外一个函数！

``` 
typealias IntFunction = Int -> Int

func map(f: IntFunction, _ g: IntFunction) -> IntFunction {
  return { x in f(g(x)) }
}

let foo = map({ $0 + 2 }, { $0 + 3 })
foo(10)
// => 15

```

So functions are Functors too! When you use fmap on a function, you’re just doing function composition!


所以函数也是 Functor！当你对函数使用 fmap，你只是在做函数组装！

## Applicatives

Applicatives take it to the next level. With an applicative, our values are wrapped in a context, just like Functors:


Applicative 将提升到另一个层级。使用 applicative，我们的值被封装在一个上下文中，就像 Functor：

![](http://adit.io/imgs/functors/value_and_context.png)

But our functions are wrapped in a context too!


但是我们的函数也被封装在一个上下文中！

![](http://adit.io/imgs/functors/function_and_context.png)

Yeah. Let that sink in. Applicatives don’t kid around. Unlike Haskell, Swift doesn't have *yet* a built-in way to deal with Applicative. But it is very easy to add one! We can define an `apply` function for every type supporting Applicative, which knows how to apply a function wrapped in the context of the type to a value wrapped in the same context:


我们继续深入，applicative 不是开玩笑的。不同于 Haskell，Swift 还并没有内置处理 applicative 的方法。但是添加一个非常简单，我们可以定义一个 `apply` 函数来支持各种类型，从而支持 applicative，applicative 知道如何将一个封装在上下文之中的函数作用于封装在同样上下文之中的值：

``` 
extension Optional {
  func apply<U>(f: (T -> U)?) -> U? {
    switch f {
      case .Some(let someF): return self.map(someF)
      case .None: return .None
    }
  }
}

extension Array {
  func apply<U>(fs: [Element -> U]) -> [U] {
    var result = [U]()
      for f in fs {
        for element in self.map(f) {
          result.append(element)
        }
      }
      return result
    }
}

```

If both `self` and the function are `.Some`, then the function is applied to the unwrapped option, otherwise `.None` is returned. *Also note that because the optional type is defined in terms of `Optional<T>` we only need to specify the generic type U in applys signature.*


如果 `self` 和函数都是 `.Some`，那么函数将被应用于解包的值，否者，`.None` 被返回。*注意因为 optional 类型是被定义为`Optional<T>`，我们只需要在 `apply` 声明处声明泛型 `U`*

We can also define `<*>`, to do the same thing:


我们也可以定义 `<*>`，做同样的使用：

``` 
infix operator <*> { associativity left }

func <*><T, U>(f: (T -> U)?, a: T?) -> U? {
  return a.apply(f)
}

func <*><T, U>(f: [T -> U], a: [T]) -> [U] {
  return a.apply(f)
}

```

![](http://adit.io/imgs/functors/applicative_just.png)

i.e:

例子：

``` 
Optional.Some({ $0 + 3 }) <*> Optional.Some(2)
// => 5

```

Using `<*>` can lead to some interesting situations. For example:

使用 `<*>` 可以产生有趣的情况，比如：

``` 
[ { $0 + 3 }, { $0 * 2 } ] <*> [1, 2, 3]
// => [ 4, 5, 6, 2, 4, 6 ]

```

![](http://adit.io/imgs/functors/applicative_list.png)

> **Note:** the original article now shows how Applicatives are more powerful than Functors in that they allow function application with multiple parameters. Again this is not feasible in vanilla Swift, but we can work around it by defining the function we want to handle in a [curried way](https://en.wikipedia.org/wiki/Currying).
> 
> **注意:**Haskell 版的原文章展示了 applicative 比 functor 强大，它允许函数应用多个参数。而这个对于 Swift 是不可行的，但是我们可以通过使用[柯里函数 (Currying)](https://en.wikipedia.org/wiki/Currying)来定义我们想要的函数。

Here’s something you can do with Applicatives that you can’t do with Functors. How do you apply a function that takes two arguments to two wrapped values?

这里是一些可以用 applicative 而不能使用 functor 的例子。如何应用一个有两个参数的函数到两个封装好的值？

``` 
func curriedAddition(a: Int)(b: Int) -> Int {
  return a + b
}

curriedAddition <^> Optional(2) <^> Optional(3)
// => COMPILER ERROR: Value of optional type '(Int -> Int)? not unwrapped; did you mean to use '!' or '??'

```

Applicatives:

``` 
curriedAddition <^> Optional(2) <*> Optional(3)

```

`Applicative` pushes `Functor` aside. “Big boys can use functions with any number of arguments,” it says. “Armed with `<^>` and `<*>`, I can take any function that expects any number of unwrapped values. Then I pass it all wrapped values, and I get a wrapped value out! AHAHAHAHAH!”

`Applicative` 把 `Functor` 推到一旁，说，“大男孩可以使用 function 处理多个参数。坐拥 `<^>` 和 `<*>`，我可以生成任意函数处理多个解包的值，然后然后将所有值打包，输出一个封装的值，哈哈哈！”

``` 
func curriedTimes(a: Int)(b: Int) -> Int {
  return a * b
}

curriedTimes <^> Optional(5) <*> Optional(3)

```

## Monads

How to learn about Monads:

1. Get a PhD in computer science.
2. Throw it away because you don’t need it for this section!

Monads add a new twist.

Functors apply a function to a wrapped value:

如何学习 Monads：

1.获得计算机博士学位；

2.不用管它，因为在本章节你并不需要！

Monads 添加一种新的方式。

Functor 为封装的值应用一个函数：

![](http://adit.io/imgs/functors/fmap.png)

Applicatives apply a wrapped function to a wrapped value:

Applicatives 为封装的值应用一个封装的函数：

![](http://adit.io/imgs/functors/applicative.png)

Monads apply a function that returns a wrapped value to a wrapped value. Monads have a function `|` (>>= in Haskell) (pronounced “bind”) to do this.


Monads have a function `flatMap` (`liftM` in Haskell) to do this. And we can define an infix operator `>>-` (`>>=` in Haskell) for it.


Monads 为封装的值，应用一个返回封装值的函数。Monads 有个函数 `|`（在 Haskell 为 >>=）（发音为 "bind"）来处理这个。

Monads 有个函数 `flatMap`（在 Haskell 为 `liftM`）能处理这个。那我们可以给它定义一个 infix 操作符 `>>-` （在 Haskell 为`>>=`）。

``` 
infix operator >>- { associativity left }

func >>-<T, U>(a: T?, f: T -> U?) -> U? {
  return a.flatMap(f)
}

```

> **Note:** Unlike `<$>`, `>>=` would compile. I decided to use `>>-` to be in line with the library [Runes](https://github.com/thoughtbot/Runes)which provides "Infix operators for monadic functions in Swift", and it's hopefully going to become the standard for this sort of things.

Let’s see an example. Good ol’ Optional is a monad:

>**注意：**不像 `<$>`, `>>=` 可以编译。我决定用 `>>-` 是由于这个库 [Runes](https://github.com/thoughtbot/Runes)，它提供在 Swift 中的 monadic 函数操作符，这很有可能在未来会成为一个标准。

![](http://adit.io/imgs/functors/context.png)

Just a monad hanging out

Suppose `half` is a function that only works on even numbers:

只是个 monad

假定 `half` 是一个函数，只能处理基本数值类型：


``` 
func half(a: Int) -> Int? {
  return a % 2 == 0 ? a / 2 : .None
}

```

![](http://adit.io/imgs/functors/half.png)

What if we feed it a wrapped value?

如果要让处理封装的值呢？

![](http://adit.io/imgs/functors/half_ouch.png)

We need to use `>>-` (`>>=` in Haskell) to shove our wrapped value into the function. Here’s a photo of `>>-`:

我们需要使用 `>>-`（在 Haskell 为 `>>=`）将封装的值强塞到这个函数里。这里是 `>>-` 的图片：

![](http://adit.io/imgs/functors/plunger.jpg)

Here’s how it works:

这里是它如何工作：

``` 
Optional(3) >>- half
// .None
Optional(4) >>- half
// 2
Optional.None >>- half
// .None

```

What's happening inside? Let's look at `>>-`'s (`>>=` in Haskell) signature again:

内部发生了什么？让我们来看下 `>>-` 的声明：

``` 
// For Optional
func >>-<T, U>(a: T?, f: T -> U?) -> U?

// For Array
func >>-<T, U>(a: [T], f: T -> [U]) -> [U]

```

![](http://adit.io/imgs/functors/bind_def.png)

So `Optional` is a Monad. Here it is in action with a `.Some(3)`!

因此 `Optional` 是一个 Monad。这里是 `.Some(3)` 的处理过程！

![](http://adit.io/imgs/functors/monad_just.png)

And if you pass in a `.None` it’s even simpler:

如果你传得时 `.None`，它会更简单：

![](http://adit.io/imgs/functors/monad_nothing.png)

You can also chain these calls:

你还可以将调用连接起来：

``` 
Optional(20) >>- half >>- half >>- half
// => .None

```

![](http://adit.io/imgs/functors/monad_chain.png)

> Note: the original article now describes Haskell's `IO` Monad. Swift doesn't have anything like that so this translation skips it.

> 注意：原文章描述 Haskell 的 `IO`Monad。Swift 并没有这个，所以跳过。


## Conclusion

## 总结

1. A functor is a type that implements `map`.
2. An applicative is a type that implements `apply`.
3. A monad is a type that implements `flatMap`.
4. `Optional` implements `map` and `flatMap`, plus we can extend it to implement `apply`, so it is a functor, an applicative, and a monad.

What is the difference between the three?

1. functor 是一种实现了 `map` 的数据类型；
2. applicative 是一个种实现了 `apply` 的数据类型；
3. monad 是一种实现了 `flatMap` 的数据类型
4. `Optional` 实现了 `map` 和 `flatMap`，加上我们可以实现 `apply` 来拓展，因此它是一个 functor，applicative, 和 monad。

那么它们三者的区别是什么呢？ 


![](http://adit.io/imgs/functors/recap.png)

- **functors**: you apply a function to a wrapped value using `map`.
- **applicatives**: you apply a wrapped function to a wrapped value using `apply`, if defined.
- **monads**: you apply a function that returns a wrapped value, to a wrapped value using`flatMap`.

- **functors**:通过 `map` 对封装的值使用了函数.
- **applicatives**: 通过使用 `apply` 对封装的值使用封装了的函数，如果你定义了的话.
- **monads**: 使用一个返回封装的值的函数，放到 `flatMap` 中处理，返回一个封装后的值.