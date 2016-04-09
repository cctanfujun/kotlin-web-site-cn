---
type: doc
layout: reference
category: "Syntax"
title: "泛型"
---

# 泛型

与Java相似，Kotlin中的类也具有类型参数，如：

``` kotlin
class Box<T>(t: T) {
  var value = t
}
```

一般而言，创建类的实例时，我们需要声明参数的类型，如：

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

但当参数类型可以从构造函数参数等途径推测时，在创建的过程中可以忽略类型参数：

``` kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```

## 协变Variance

Java的变量类型中，最为精妙的是通配符（wildcards）类型(详见 [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html))。
但是Kotlin中并不具备该类型，替而代之的是：声明设置差异（declaration-site variance）与类型推测。

首先，我们考虑一下Java中的通配符（wildcards）的意义。该问题在文档 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 28: *Use bounded wildcards to increase API flexibility*中给出了详细的解释。
首先，Java中的泛型类型是不变的，即`List<String>`并不是`List<Object>`的子类型。
原因在于，如果List是可变的，并不会
优于Java数组。因为如下代码在编译后会产生运行时异常：

``` java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! The cause of the upcoming problem sits here. Java prohibits this!
objs.add(1); // Here we put an Integer into a list of Strings
String s = strs.get(0); // !!! ClassCastException: Cannot cast Integer to String
```
因此，Java规定泛型类型不可变来保证运行时的安全。但这样规定也具有一些影响。如， `Collection`接口中的`addAll()`
方法，该方法的签名应该是什么？直观地，我们这样定义：

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

但随后，我们便不能实现以下肯定安全的事：

``` java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! Would not compile with the naive declaration of addAll:
                   //       Collection<String> is not a subtype of Collection<Object>
}
```

（更详细的解析参见[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 25: *Prefer lists to arrays*）


以上正是为什么`addAll()`的方法签名如下的原因：

``` java
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```

**通配符类型（wildcard**）的声明 `? extends T`表明了该方法允许一类对象是 `T`的*子类型*，而非必须得是 `T`本身。
这意味着我们可以安全地从元素（ T的子类集合中的元素）**读取** `T`，同时由于
我们并不知道 `T`的子类型，所以**不能写**元素。
反过来，该限制可以让`Collection<String>`表示为`Collection<? extends Object>`的子类型。
简而言之，带**extends**限定（上限）的通配符类型（wildcard）使得类型是**协变的（covariant）**。

理解为什么这样做可以使得类型的表达更加简单的关键点在于：如果只能从集合中获取元素，那么就可以使用 `String`集合，
从中读取`Object`也没问题 。反过来，如果只能向集合中_放入_元素，就可以用
`Object`集合并向其中放入`String`：在Java中有`List<? super String>`是`List<Object>`的**超类**。

后面的情况被称为**“逆变性”（contravariance）**，这种性质是的只可以调用方法时利用String为`List<? super String>` 的参数。
（例如，可以调用`add(String)`或者`set(int, String)`),或者
当调用函数返回`List<T>`中的`T`，你获取的并非一个`String`而是一个 `Object`。

Joshua Bloch成这类为只可以从**Producers(生产者)**处 **读取**的对象，以及只可向**Consumers（消费者）**处**写**的对象。他表示：“*为了最大化地保证灵活性，在输入参数时使用通配符类型来代表生产者或者消费者*”，同时他也提出了以下术语：

*PECS 代表生产者扩展，消费者超类（roducer-Extends, Consumer-Super）。*

*注记*：当使用一个生产者对象时，如`List<? extends Foo>`，在该对象上不可调用 `add()` 或 `set()`方法。但这不代表
该对象是**不变的**。例如，可以调用 `clear()`方法移除列表里的所有元素，因为 `clear()`方法
不含任何参数。通配符类型唯一保证的仅仅是**类型安全**。不可变性完全是另一个话题了。

### 声明处协变

假设有一个泛型接口`Source<T>` ，该接口中不存在将 `T` 作为参数的方法，只有返回值为 `T` 的方法：

``` java
// Java
interface Source<T> {
  T nextT();
}
```

那么，利用`Source<Object>`类型的对象向 `Source<String>` 实例中存入引用是极为安全的，因为不存在任何可以调用的消费者方法。但是Java并不知道这点，依旧禁止这样操作：

``` java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Not allowed in Java
  // ...
}
```

为了修正这一点，我们需要声明对象的类型为`Source<? extends Object>`，有一点无意义，因为我们可以像以前一下在该对象上调用所有相同的方法，所以复杂类型没有增加值。但是编译器并不可以理解。

在Kotlin中，我们有一种途径向编译器解释该表达，称之为：**声明处协变**：我们可以标注源的**变量类型**为`T` 来确保它仅从`Source<T>`成员中**返回**（生产），并从不被消费。
为此，我们提供**输出**修改：

``` kotlin
abstract class Source<out T> {
  abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
  val objects: Source<Any> = strs // This is OK, since T is an out-parameter
  // ...
}
```

常规是：当一个类`C`中类型为`T`的变量被声明为**输出**，它将仅出现于类`C`的成员**输出**\-位置，反之使得`C<Base>`可以安全地成为
`C<Derived>`的超类。

简而言之，称类`C`是参数`T`中的**协变的**，或`T`是一个**协变**的参数类型。
我们可以认为`C`是`T`的一个生产者，同时不是`T`的消费者。

**out**修饰符叫做**协变注解**，同时由于它在参数类型位置被提供，所以我们讨论**声明处协变**。
与Java的**使用处协变**相反，类型使用通配符使得类型协变。

另外除了**out**，Kotlin又补充了一项协变注释：**in**。它是的变量类型**反变**：只可以被消费而不可以
被生产。反变类的一个很好的例子是 `Comparable`：

``` kotlin
abstract class Comparable<in T> {
  abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
  x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
  // Thus, we can assign x to a variable of type Comparable<Double>
  val y: Comparable<Double> = x // OK!
}
```

我们认为**in**和**out**是自解释（他们早已成功地被应用于C#中），
所以上文的解释并非必须的，并且读者可以从

**[The Existential](http://en.wikipedia.org/wiki/Existentialism) Transformation: Consumer in, Producer out\!** 中获取更加深入的理解:)。

## 类型预测

### 使用处协变：类型预测

声明变量类型T为*out*是极为方便的，并且在运用子类型的过程中也没有问题。是的，当该类**可以**被仅限于返回`T`，但是如果**不可以**呢？
一个很好的例子是 Array：

``` kotlin
class Array<T>(val size: Int) {
  fun get(index: Int): T { /* ... */ }
  fun set(index: Int, value: T) { /* ... */ }
}
```

该类中的`T`不仅不可以被co\- 也不能被逆变。这造成了极大的不灵活性。考虑该方法：

``` kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
  assert(from.size == to.size)
  for (i in from.indices)
    to[i] = from[i]
}
```

该方法试图从一个数组中copy元素到另一个数组。我们尝试着在实际中运用它：

``` kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3)
copy(ints, any) // Error: expects (Array<Any>, Array<Any>)
```

这里，我们陷入了一个类似的问题：`Array<T>`中的`T`是**不变**的，所以不论是`Array<Int>`或`Array<Any>`
都不是另一个的子类型。为什么？因为copy操作**可能**是不安全的行为，例如，它可能尝试向来源**写**一个String，
如果我们真的将其转换为`Int`数组，随后 `ClassCastException`异常可能会被抛出。

那么，我们唯一需要确保的是`copy()`不会执行任何不安全的操作。我们试图阻止它向来源**写**，我们可以：

``` kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

我们称该做法为**类型预测**：`源`并不仅是一个数组，并且可以要是可预测的。我们仅可调用返回类型为
`T`的方法，如上，我们只能调用`get()`方法。这就是我们使用**使用位置可变性**而非Java中的`Array<? extends Object>`的
更加明确简单的方法

同时，你也可以利用**in**预测输入类型：

``` kotlin
fun fill(dest: Array<in String>, value: String) {
  // ...
}
```

`Array<in String>` 比对于Java中的`Array<? super String>`, 例如，你可以向`fill()`方法传递一个 `CharSequence` 数组或者一个 `Object`数组。

### 星-预测 Star-Projections

有时，你试图说你并不知道任何类型声明的方法，但是仍旧想安全地使用他。
这里的安全方法指我们需要对*out*\-预测

Kotlin提供了所谓的**星预测**语法如下：

 - For `Foo<out T>`, where `T` is a covariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>`. It means that when the `T` is unknown you can safely *read* values of `TUpper` from `Foo<*>`.
 - For `Foo<in T>`, where `T` is a contravariant type parameter, `Foo<*>` is equivalent to `Foo<in Nothing>`. It means there is nothing you can *write* to `Foo<*>` in a safe way when `T` is unknown.
 - For `Foo<T>`, where `T` is an invariant type parameter with the upper bound `TUpper`, `Foo<*>` is equivalent to `Foo<out TUpper>` for reading values and to `Foo<in Nothing>` for writing values.

If a generic type has several type parameters each of them can be projected independently.
For example, if the type is declared as `interface Function<in T, out U>` we can imagine the following star-projections:

 - `Function<*, String>` means `Function<in Nothing, String>`;
 - `Function<Int, *>` means `Function<Int, out Any?>`;
 - `Function<*, *>` means `Function<in Nothing, out Any?>`.

*注记*：星预测很像Java中的raw类型，但是比raw类型更加安全。

# 泛型函数

不仅仅是类可以类型参数，函数也可以有。类型参数放置在函数名前：

``` kotlin
fun <T> singletonList(item: T): List<T> {
  // ...
}

fun <T> T.basicToString() : String {  // extension function
  // ...
}
```

如果参数类型是从调用方显式传递而来，那么它要在函数名之后被声明：

``` kotlin
val l = singletonList<Int>(1)
```

# 泛型约束

集合的所有可能类型可以被给定的被约束的**泛型约束**参数类型替代。

## 上界

约束最常见的类型是**上界**相比较于Java中的*extends*关键字：

``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
  // ...
}
```

在冒号之后被声明的是**上界**：代替`T`的仅可为 `Comparable<T>`的子类型。例如：

``` kotlin
sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

默认的上界（如果没有声明）是`Any?`。只能有一个上界可以在尖括号中被声明。
如果相同的类型参数需要多个上界，我们需要分割符 **where**\-子句，如:

``` kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```


