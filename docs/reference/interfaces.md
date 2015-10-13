---
type: doc
layout: reference
category: "Syntax"
title: "Interfaces"
---

# 接口

Kotlin中的接口和Java 8中的接口类似，它们可以包含抽象方法的声明以及实现。与抽象类的不同之处在于接口不能存储状态。它们可以拥有一些抽象的属性。

使用*interface*{: .keyword }定义一个接口

``` kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

## 实现接口

一个类或对象可以实现一个或多个接口

``` kotlin
class Child : MyInterface {
   fun bar() {
      // body
   }
}
```

## 接口的属性

接口是没有状态的，所以只允许定义一些无状态的属性。

``` kotlin
interface MyInterface {
    val property: Int // abstract

    fun foo() {
        print(property)
    }
}

class Child : MyInterface {
    override val property: Int = 29
}
```

## 解决覆写时候的冲突

当我们在父级列表中声明多种类型，可能会出现继承同一个方法的多种实现。例如：

``` kotlin
interface A {
  fun foo() { print("A") }
  fun bar()
}

interface B {
  fun foo() { print("B") }
  fun bar() { print("bar") }
}

class C : A {
  override fun bar() { print("bar") }
}

class D : A, B {
  override fun foo() {
    super<A>.foo()
    super<B>.foo()
  }
}
```

接口*A*和*B*同时声明了*foo()*和*bar()*两个方法。它们都实现了*foo()*方法，但是只有*B*实现了*bar()*方法（*bar()*不是*A*中标记的那个抽象方法，因为如果没有方法体，就是接口的默认方法）。现在，如果我们从*A*派生出一个具体类*C*，我们必须覆写并实现*bar()*方法。如果我们从*A*和*B*共同派生类*D*，可以不覆写*bar()*方法，因为我们已经继承了一个该方法的实现，但是同时我们也继承了*foo()*方法的两种实现，编译器不知道选择哪一个，此时就会要求我们必须覆写*foo()*方法并告知明确需要使用哪一个。

------------

翻译 by [whilu](https://github.com/whilu) 
