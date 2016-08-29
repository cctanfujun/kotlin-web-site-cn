---
type: doc
layout: reference
category: "Syntax"
title: "委托属性"
---

# 委托属性

有一些种类的属性，虽然我们可以在每次需要的时候手动实现它们，但是如果能够把他们只实现一次
并放入一个库同时又能够一直使用它们那会更好。例如：

* 延迟属性（lazy properties）: 数值只在第一次被访问的时候计算。
* 可观察属性（observable properties）: 监听器得到关于这个特性变化的通知，
* 把所有属性储存在一个map中，而不是每个在单独的字段里。

为了支持这些(或者其他)例子，Kotlin 采用 _委托属性_:

``` kotlin
class Example {
  var p: String by Delegate()
}
```

语法是: `val/var <property name>: <Type> by <expression>`.在*by*{:.keyword}后面的表达式是 _委托_,
因为 `get()` (和 `set()`) 相当于属性会被委托给它的 `getValue()` 和 `setValue()` 方法。
特性委托不必实现任何的接口，但是需要提供一个 `getValue()`函数（和 `setValue()` --- 对于 *var*{:.keyword}'s）。
例如:

``` kotlin
class Delegate {
  operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
    return "$thisRef, thank you for delegating '${property.name}' to me!"
  }

  operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
    println("$value has been assigned to '${property.name} in $thisRef.'")
  }
}
```

当我们读取一个`Delegate`的委托实例 `p` , `Delegate`中的`getValue()`就被调用,
所以它第一变量就是我们从 `p` 读取的实例,第二个变量代表 `p` 自身的描述。
(例如你可以用它的名字). 下面是例子:

``` kotlin
val e = Example()
println(e.p)
```

打印结果：

```
Example@33a17727, thank you for delegating ‘p’ to me!
```

类似的，当我们给 `p` 赋值, `setValue()` 函数就被调用. 前两个参数是一样的，第三个参数保存着将要被赋予的值:

``` kotlin
e.p = "NEW"
```

打印结果：

```
NEW has been assigned to ‘p’ in Example@33a17727.
```

## 属性委托要求

这里我们总结委托对象的要求。

对于一个 **只读** 属性 (如 *val*{:.keyword}), 一个委托一定会提供一个 `getValue`函数来获取下面的参数:

* 接收者 --- 必须与_属性所有者_类型相同或者是其父类(对于扩展属性，类型范围允许扩大),
* 包含数据 --- 一定要是 `KProperty<*>` 的类型或它的父类型,

这个函数必须返回同样的类型作为属性（或者子类型）

对于一个 **可变** 属性 (如 *var*{:.keyword}), 一个委托需要额外地提供一个函数 `setValue` 来获取下面的参数:

* 接收者 --- 同 `getValue()`,
* 包含数据 --- 同 `getValue()`,
* 新的值 --- 必须和属性同类型或者是他的父类型。

`getValue()` 或/和 `setValue()` 函数可能会作为代理类的成员函数或者扩展函数来提供。
当你需要代理一个属性给一个不是原来就提供这些函数的对象的时候，后者更为方便。
两种函数都需要用`operator`关键字来进行标记


## 标准委托

标准库中对于一些有用的委托提供了工厂（factory）方法。

### 延迟属性 Lazy

函数 `lazy()` 接受一个 lambda 然后返回一个可以作为实现延迟属性的委托 `Lazy<T>` 实例来:
第一次对于 `get()`的调用会执行（之前）传递到 `lazy()`的lamda表达式并记录结果,
后面的 `get()` 调用会直接返回记录的结果。


``` kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
```

默认地，对于lazy属性的计算是**同步锁（synchronized）** 的: 这个值只在一个线程被计算，并且所有的线程会看到相同的值。如果初始化代理的同步锁不是必须的，以至于多个线程可以同步地执行，那么将`LazyThreadSafetyMode.PUBLICATION`作为一个变量传递给`lazy()`函数。而且如果你确定初始化将总是发生在单个线程，那么你可以使用 `LazyThreadSafetyMode.NONE` 模式, 它不会有任何线程安全的保证和相关的开销


### 可观察属性 Observable

`Delegates.observable()` 需要两个参数：初始值和handler。
这个 handler 会在每次我们给赋值的时候被调用 (在工作完成前).
它有三个参数:一个被赋值的属性，旧的值和新的值：

``` kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

这个例子输出：

```
<no name> -> first
first -> second
```

如果你想有能力来截取和“否决”它分派的事件，就使用 `vetoable()` 取代 `observable()`.
被传递给 `vetoable` 的handler会在属性被赋新的值_之前_执行

## 把属性储存在 Map 中

一个参加的用例是在一个map里存储属性的值。
这经常出现在解析JSON或者做其他的“动态”的事情应用里头。
在这样的情况下，你需要使用map的实例本身作为代理用于代理属性

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

在这个例子中，构造函数会接受一个map参数：

``` kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

委托会从这个map中取值 (通过string类型的key，就是属性的名字):


``` kotlin
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

对于 *var*{:.关键词}的变量，我们可以把只读的`Map`换成 `MutableMap`就可以了

``` kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```
