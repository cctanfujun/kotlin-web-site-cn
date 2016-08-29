---
type: doc
layout: reference
category: "Syntax"
title: "高阶函数和lambda表达式"
---

# 高阶函数和lambda表达式

## 高阶函数

高阶函数是一种能用函数作为参数或者返回值为函数的一种函数。
`lock()`是高阶函数中一个比较好的例子，它接受一个lock对象和一个函数，获得锁，运行传入的函数，并释放锁：

``` kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
  lock.lock()
  try {
    return body()
  }
  finally {
    lock.unlock()
  }
}
```

我们分析一下上面的代码：函数`body`拥有[函数类型](#function-types):`() -> T`
所以body应该是一个不带参数并且返回`T`类型的值的函数。
它在*try*{: .keyword }代码块中调用，被`lock`保护的，当`lock()`函数被调用时返回他的值。

如果我们想调用`lock()`函数，我们可以把另一个函数传递给它作为参数(详见 [函数引用](reflection.html#function-references)):

``` kotlin
fun toBeSynchronized() = sharedResource.operation()

val result = lock(lock, ::toBeSynchronized)
```

另外一种更为便捷的方式是传入一个 [lambda 字面量](#lambda-expressions-and-anonymous-functions):

``` kotlin
val result = lock(lock, { sharedResource.operation() })
```

lambda 表达式 [这里](#lambda-expressions-and-anonymous-functions)有更详细的描述, 但是为了继续这一段，让我们看到一个简短的概述：

* 一个 lambda 表达式总是被大括号包围着。
* 其参数（如果有的话）被声明在`->`之前（参数类型可以省略）
* 函数体在 `->` 后面 (如果存在的话).

在Kotlin中, 如果函数的最后一个参数是一个函数，那么该参数可以在括号外指定：

``` kotlin
lock (lock) {
  sharedResource.operation()
}
```

另一个高阶函数的例子是 `map()` ( [MapReduce](http://en.wikipedia.org/wiki/MapReduce)):

``` kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
  val result = arrayListOf<R>()
  for (item in this)
    result.add(transform(item))
  return result
}
```

这个函数可以如下调用:

``` kotlin
val doubled = ints.map { it -> it * 2 }
```

Note that the parentheses in a call can be omitted entirely if the lambda is the only argument to that call.

还有一个有用的公约是：如果函数字面量有一个参数，
那它的声明可以省略（连同 `->`），用 `it` 表示。

``` kotlin
ints.map { it * 2 }
```

这些约定可以写成 [LINQ-风格](http://msdn.microsoft.com/en-us/library/bb308959.aspx) 的代码:

``` kotlin
strings filter {it.length == 5} sortBy {it} map {it.toUpperCase()}
```

## 内联函数

使用[内联函数](inline-functions.html)有时能提高高阶函数的性能。

## Lambda 表达式和匿名函数

一个 lambda 表达式货匿名函数是一个“函数字面值”, 即 一个未声明的函数，
但却立即写为表达式。思考下面的例子：

``` kotlin
max(strings, { a, b -> a.length < b.length })
```

`max`函数是一个高阶函数, 也就是说 他的第二个参数是一个函数.
这个参数是一个表达式，但它本身也是一个函数, 也就是函数字面量.写成一个函数的话，它相当于

``` kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 函数类型

对于一个接受一个函数作为参数的函数，我们必须为该参数指定一个函数类型。
譬如上述`max`函数定义如下：

``` kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
  var max: T? = null
  for (it in collection)
    if (max == null || less(max, it))
      max = it
  return max
}
```

参数 `less` 是一个 `(T, T) -> Boolean`类型的函数, 也就是说`less`函数接收两个`T`类型的参数并返回一个`Boolean`值:
如果第一个比第二个小就返回`True`.

在第四行代码里, `less` 被用作为一个函数: 它传入两个`T`类型的参数.

如上所写的是就函数类型, 或者还有命名参数, 如果你想文档化每个参数的含义。

``` kotlin
val compare: (x: T, y: T) -> Int = ...
```

### Lambda表达式语法

Lambda 表达式的全部语法形式, 也就是函数类型的字面量, 譬如下面的代码:

``` kotlin
val sum = { x: Int, y: Int -> x + y }
```

一个函Lambda表达式总是被大括号包围着，
在括号内有全部语法形式中的参数声明并且有可选的类型注解，
函数体后面有一个 `->` 符号。
如果我们把所有的可选注解都留了出来，那剩下的是什么样子的：

``` kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

这是非常常见的，一个 lambda 表达式只有一个参数。
如果Kotlin能自己计算出自己的数字签名，我们就可以不去声明这个唯一的参数。并且用
`it`进行隐式声明。

``` kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

请注意，如果函数取另一个函数作为最后一个参数，该 lambda 表达式参数可以放在
括号外的参数列表。
语法细则详见 [callSuffix](grammar.html#call-suffix).

### 匿名函数

上述 lambda 表达式的语法还少了一个东西： 能够指定函数的返回
类型。在大多数情况下, 这是不必要的。因为返回类型可以被自动推断出来.
然而，如果你需要要明确的指定。你需要一个替代语法:_匿名函数_.

``` kotlin
fun(x: Int, y: Int): Int = x + y
```

匿名函数看起来很像一个普通函数声明, 只是名字被省略了。内容
也是一个表达式（如上面的代码）或者代码块:

``` kotlin
fun(x: Int, y: Int): Int {
  return x + y
}
```

指定的参数和返回类型与指定一个普通函数方式相同，只是如果参数
类型能沟通过上下文推断出来，那么该参数类型是可以省略的:

``` kotlin
ints.filter(fun(item) = item > 0)
```

匿名函数的返回类型推断法只适用于常规函数：具有表达式体并且必须明确指定函数体有代码（或者假定`Unit`）块的返回类型能够被自动推断出来。

请注意，匿名函数参数始终在圆括号内传递。允许在
函数括号外使用的速记语法只针对于 lambda 函数。

另一个 lambda 表达式和匿名函数区别是
[non-local    returns](inline-functions.html#non-local-returns)的行为。一个不带标签的*return*{：. keyword} 语句
总是在用*fun*{: .keyword } 关键词声明的函数中返回。这意味着 lambda 表达式中的return*{: .keyword }
将在函数闭包中返回 。然而匿名函数*return*{: .keyword}的就是在匿名函数自身中返回。

### 闭包

一个 lambda 表达式或者匿名函数（以及一个[本地函数](functions.html#local-functions)本地函数和一个 [对象表达式](object-declarations.html#object-expressions)）
可以访问他的_闭包_,即声明在外范围内的变量。与java不同，在闭包中捕获的变量可以被修改：

``` kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
  sum += it
}
print(sum)
```


### 带接收者得函数字面值

kotlin提供了使用一个特定的 _receiver对象_ 来调用一个函数的能力.
在函数体内部，你可以调用 接受者对象 的方法而不需要任何额外的限定符。
这和 扩展函数 有点类似，它允你在函数体内访问接收器对象的成员。

他们的一个最重要的例子是[Type-safe Groovy-style builders](type-safe-builders.html)的使用。

这样的函数字面量的类型是一个带receiver的函数类型

``` kotlin
sum : Int.(other: Int) -> Int
```
如果函数是一个在receiver对象上的方法，那么这个函数可以被调用

``` kotlin
1.sum(2)
```

匿名函数的语法允许你直接指定函数的receiver的类型
如果你需要用一个receiver声明一个函数类型的变量，并且在后面用到它，那么这个语法就很有用

``` kotlin
val sum = fun Int.(other: Int): Int = this + other
```

当receiver类型能够被从上下文推断的时候，Lamda表达式能够被用于带receiver的函数字面量

``` kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
  val html = HTML()  // create the receiver object
  html.init()        // pass the receiver object to the lambda
  return html
}


html {       // lambda with receiver begins here
    body()   // calling a method on the receiver object
}
```
