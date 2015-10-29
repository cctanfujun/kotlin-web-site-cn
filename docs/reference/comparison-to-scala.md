---
type: doc
layout: reference
category: FAQ
title: "Comparison to Scala"
---

# 对比Scala

设计Kotlin语言有两主要的目的:

* 使编译至少和Java一样快
* 保证表达到位的同时，语言足够简洁

在这考虑一下, 如果你的Scala已相当得心应手, 或许你不需要再学习Kotlin

## Scala有什么Kotlin没有的

* 隐式转换,标量参数, 等等
    * 在Scala中, 由于画面中有太多的隐式转换，有时不使用debugger，会很难弄清楚code具体发生了什么
    * 为了达到功能的多样性，在Kotlin中要使用[扩展函数](extensions.html).
* 可重写的成员类型
* 路径依赖类型
* 宏
* 存在类型
    * [类型预测](generics.html#type-projections) 是一种非常特殊的情况
* 特性初始化的复杂逻辑
    * 参照 [类与继承](classes.html)
* 自定义符号运算
    * 参照 [操作符重载](operator-overloading.html)
* 嵌入式XML
    * 参照 [类型安全 Groovy风格 builders](type-safe-builders.html)

未来Kotlin中可能要增加的:

* 结构类型
* Value类型
* Yield operator
* Actors
* 并行集合

## Kotlin有什么Scala没有的

* [零开销 空安全](null-safety.html)
    * Scala有“语法和运行时包装”选项
* [智能转换](typecasts.html)
* [Kotlin的内联函数助力非局部跳跃](inline-functions.html#inline-functions)
* [第一类 授权](delegation.html). 同时通过实施第三方插件: Autoproxy
