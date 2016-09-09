---
type: doc
layout: reference
category: "Syntax"
title: "扩展"
---

# 扩展

Kotlin和c#、Gosu一样，能够扩展一个类的新功能，而无需继承类或使用任何类型的设计模式，如装饰者。
这通过特殊的声明叫做_extensions_。Kotlin支持_extension functions_ 和 _extension properties_.

## 扩展函数

声明一个扩展函数，我们需要用一个 _接收者类型_ 也就是被扩展的类型来作为他的前缀。
下面是为`MutableList<Int>`添加一个`swap`方法：

``` kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
  val tmp = this[index1] // 'this' corresponds to the list
  this[index1] = this[index2]
  this[index2] = tmp
}
```

这个*this*{: .keyword }关键字在扩展方法内接受对应的对象（在点符号以前传过来的）
现在，我们可以像一个其他方法一样调用`MutableList<Int>`:  

``` kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```

当然，这个方法像这样`MutableList<T>`，我们可以使用泛型：

``` kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
  val tmp = this[index1] // 'this' corresponds to the list
  this[index1] = this[index2]
  this[index2] = tmp
}
```

在接收类型表达式中，我们要在方法名可用前声明泛型类型参数。
参见[Generic functions](generics.html).

## 扩展的静态解析

扩展不能真正的修改他们继承的类。通过定义一个扩展，你不能在类内插入新成员，
仅仅是通过该类的实例用点表达式去调用这个新函数。

我们想强调下扩展方法是被静态分发的，即他们不是接收类型的虚方法。
This means that the extension function being called is determined by the type of the expression on which the function is invoked,
not by the type of the result of evaluating that expression at runtime. For example:

``` kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

This example will print "c", because the extension function being called depends only on the declared type of the
parameter `c`, which is the `C` class.

如果一个类有一个成员函数和一个扩展函数，而这两个函数又有相同的名字和参数列表，这时当一个对象调用时，**成员函数总会优先**. 
如下：

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

如果我们调用`C`类型的`c`的`c.foo()`，它将打印"member"，而不是"extension".

## Nullable接收者

注意扩展可被定义为可空的接收类型。这样的扩展可以被对象变量调用，
即使他的值是null，你可以在方法体内检查`this == null`，这也允许你
在没有检查null的时候调用Kotlin中的toString()：检查发生在扩展方法的内部的时候

``` kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

## 扩展属性

和方法相似，Kotlin支持扩展属性

``` kotlin
val <T> List<T>.lastIndex: Int
  get() = size - 1
```

注意：由于扩展没有实际的将成员插入类中，因此对扩展来说是无效的
属性是有[backing field](properties.html#backing-fields).这就是为什么**初始化其不允许有
扩展属性**。他们的行为只能显式的使用 getters/setters.  

例子:

``` kotlin
val Foo.bar = 1 // error: initializers are not allowed for extension properties
```


## 伴生对象的扩展

如果一个类定义有一个[伴生对象](object-declarations.html#companion-objects) ，你也可以为伴生对象定义
扩展函数和属性

``` kotlin
class MyClass {
  companion object { }  // will be called "Companion"
}

fun MyClass.Companion.foo() {
  // ...
}
```

就像伴生对象的其他普通成员，只用用类名作为限定符去调用他们

``` kotlin
MyClass.foo()
```


## 扩展范围

大多数时候，我们定义扩张方法在顶层，即直接在包里

``` kotlin
package foo.bar

fun Baz.goo() { ... }
```

使用一个定义的包之外的扩展，我们需要导入他的头文件：

``` kotlin
package com.example.usage

import foo.bar.goo // importing all extensions by name "goo"
                   // or
import foo.bar.*   // importing everything from "foo.bar"

fun usage(baz: Baz) {
  baz.goo()
)

```

 更多信息参见[Imports](packages.html#imports)

## Declaring Extensions as Members

Inside a class, you can declare extensions for another class. Inside such an extension, there are multiple _implicit receivers_ -
objects members of which can be accessed without a qualifier. The instance of the class in which the extension is declared is called
_dispatch receiver_, and the instance of the receiver type of the extension method is called _extension receiver_.

``` kotlin
class D {
    fun bar() { ... }
}

class C {
    fun baz() { ... }

    fun D.foo() {
        bar()   // calls D.bar
        baz()   // calls C.baz
    }

    fun caller(d: D) {
        d.foo()   // call the extension function
    }
}
```

In case of a name conflict between the members of the dispatch receiver and the extension receiver, the extension receiver takes
precedence. To refer to the member of the dispatch receiver you can use the [qualified `this` syntax](this-expressions.html#qualified).

``` kotlin
class C {
    fun D.foo() {
        toString()         // calls D.toString()
        this@C.toString()  // calls C.toString()
    }
```

Extensions declared as members can be declared as `open` and overridden in subclasses. This means that the dispatch of such
functions is virtual with regard to the dispatch receiver type, but static with regard to the extension receiver type.

``` kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // call the extension function
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // prints "D.foo in C"
C1().caller(D())  // prints "D.foo in C1" - dispatch receiver is resolved virtually
C().caller(D1())  // prints "D.foo in C" - extension receiver is resolved statically
```


## Motivation

## 动机

在Java中，我们将类命名为"\*Utils": `FileUtils`, `StringUtils`等，著名的`java.util.Collections`也属于同一种命名方式。
关于这些Utils-classes的不愉快的部分是这样写代码的：

``` java
// Java
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list))
```

这些类名总是碍手碍脚的，我们可以通过静态导入得到：

``` java
// Java
swap(list, binarySearch(list, max(otherList)), max(list))
```

这会变得好一点，但是我们并没有从IDE强大的自动补全功能中得到帮助。我们希望它能更好点

``` java
// Java
list.swap(list.binarySearch(otherList.max()), list.max())
```

但是我们不希望实现`List`类内所有可能的方法，对吧？这时候扩展将会帮助我们。
