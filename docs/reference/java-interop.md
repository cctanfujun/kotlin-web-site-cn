---
type: doc
layout: reference
category: "Interop"
title: "在Kotlin中调用Java代码"
---

# 在Kotlin中调用Java代码

Kotlin 在设计时就是以与 java 交互为中心的。现存的 Java 代码可以在 kotlin 中使用，并且 Kotlin 代码也可以
在 Java 中流畅运行。这节我们会讨论在 kotlin 中调用 Java 代码的细节。

基本所有的 Java 代码都可以运行

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
  val list = ArrayList<Int>()
  // 'for'-loops work for Java collections:
  for (item in source)
    list.add(item)
  // Operator conventions work as well:
  for (i in 0..source.size() - 1)
    list[i] = source[i] // get and set are called
}
```

## Getters 和 Setters

若一个属性的getter/setter方法按照约定的规范进行命名(getter方法以`get`开头不带参数/setter方法以`set`开头
带一个参数)，那么在Kotlin中可以直接对这个属性进行访问。如：

``` kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // call getFirstDayOfWeek()
        calendar.firstDayOfWeek = Calendar.MONDAY       // call setFirstDayOfWeek()
    }
}
```

需要注意的是，对于只有setter方法的属性，kotlin目前还不支持对它的直接访问。

## 返回void的方法

如果一个Java方法返回void，那么在Kotlin中，它会返回`Unit`。
万一有人使用它的返回值，Kotlin的编译器会在调用的地方赋值，
因为这个值本身已经提前可以预知了(这个值就是`Unit`)。

## 将Java代码中与Kotlin关键字冲突的标识符进行转义

一些Kotlin的关键字在Java中是合法的标识符: *in*{: .keyword }, *object*{: .keyword }, *is*{: .keyword }, 等等.
如果一个Java库在方法中使用了Kotlin关键字,你仍然可以使用这个方法
使用反引号(`)转义来避免冲突。

``` kotlin
foo.`is`(bar)
```

## Null安全性和平台类型

Java中的所有引用都可能是*null*{: .keyword }值，这使得Kotlin严格的null控制对来自Java的对象来说变得不切实际。
在Kotlin中Java声明类型被特别对待叫做*platform types*.这种类型的Null检查是不严格的，
所以他们还维持着同Java中一样的安全性 (更多参见[下面](#mapped-types))。

考虑如下例子:

``` kotlin
val list = ArrayList<String>() // non-null (constructor result)
list.add("Item")
val size = list.size() // non-null (primitive int)
val item = list[0] // platform type inferred (ordinary Java object)
```

当我们调用平台类型的变量上的方法时，Kotlin不会在编译阶段报出可能为空的错误，
但在运行时，会产生空指针异常，或者是断言失败的错误。后者是因为kotlin为了阻止null值传播会生成非空断言语句。

``` kotlin
item.substring(1) // 允许, 如果item为空会抛异常
```

平台类型是*不可转义*的，也就是说我们不能在程序里把他们写出来。
当把一个平台数值赋值给kotlin变量的时候（变量会有一个推断出来的平台类型，
上面的例子里就是`item`的类型），我们可以用类型推断，或者指定我们期望的类型（nullable和non-null类型都可以）：

``` kotlin
val nullable: String? = item // 允许，没有问题
val notNull: String = item // 允许，运行时可能失败
```

如果我们指定了一个非空类型，编译器会在赋值前额外生成一个断言。这样Kotlin的非空变量就不会有
空值。当把平台数值传递给只接受非空数值的kolin函数的时候，也同样会生成这个断言，
编译器尽可能的阻止空置在程序里传播。（因为泛型的存在，
有时也不能百分百的阻止）

### 平台类型的概念

如上所述，平台类型不能在程序里显式的出现，
所以没有针对他们的语法。
然而，编译器和IDE有时需要显式他们(如在错误信息，参数信息中)，所以我们用
一个好记的标记来表示他们：

* `T!` 表示 "`T` 或者 `T?`"
* `(Mutable)Collection<T>!` 表示 "`T`的java集合，可变的或不可变的，可空的或非空的"
* `Array<(out) T>!` 表示 "`T`(或`T`的子类)的java数组，可空的或非空的"

### 可空性注解

Java types which have nullability annotations are represented not as platform types, but as actual nullable or non-null
Kotlin types. The compiler supports several flavors of nullability annotations, including:

  * [JetBrains](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)
(`@Nullable` and `@NotNull` from the `org.jetbrains.annotations` package)
  * Android (`com.android.annotations` and `android.support.annotations`)
  * JSR-305 (`javax.annotation`)
  * FindBugs (`edu.umd.cs.findbugs.annotations`)
  * Eclipse (`org.eclipse.jdt.annotation`)
  * Lombok (`lombok.NonNull`).

You can find the full list in the [Kotlin compiler source code](https://github.com/JetBrains/kotlin/blob/master/core/descriptor.loader.java/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt).

## 已映射类型

Kotlin特殊处理一部分java类型。这些类型不是通过as或is来直接转换，而是_映射_到了指定的kotlin类型上。
映射只发生在编译期间，运行时仍然是原来的类型。
java的原生类型映射成如下kotlin类型（记得 [平台类型](#platform-types)）：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |
{:.zebra}

一些非原生类型也会作映射：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `java.lang.Object`       | `kotlin.Any!`    |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`    |
| `java.lang.Enum`         | `kotlin.Enum!`    |
| `java.lang.Annotation`   | `kotlin.Annotation!`    |
| `java.lang.Deprecated`   | `kotlin.Deprecated!`    |
| `java.lang.Void`         | `kotlin.Nothing!`    |
| `java.lang.CharSequence` | `kotlin.CharSequence!`   |
| `java.lang.String`       | `kotlin.String!`   |
| `java.lang.Number`       | `kotlin.Number!`     |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |
{:.zebra}

集合类型在Kotlin里可以是只读的或可变的，因此Java集合类型作如下映射：
（下表所有的Kotlin类型都在`Kotlin`包里）

| **Java类型** | **Kotlin只读类型**  | **Kotlin可变类型** | **加载的平台类型** |
|---------------|------------------|----|----|
| `Iterator<T>`        | `Iterator<T>`        | `MutableIterator<T>`            | `(Mutable)Iterator<T>!`            |
| `Iterable<T>`        | `Iterable<T>`        | `MutableIterable<T>`            | `(Mutable)Iterable<T>!`            |
| `Collection<T>`      | `Collection<T>`      | `MutableCollection<T>`          | `(Mutable)Collection<T>!`          |
| `Set<T>`             | `Set<T>`             | `MutableSet<T>`                 | `(Mutable)Set<T>!`                 |
| `List<T>`            | `List<T>`            | `MutableList<T>`                | `(Mutable)List<T>!`                |
| `ListIterator<T>`    | `ListIterator<T>`    | `MutableListIterator<T>`        | `(Mutable)ListIterator<T>!`        |
| `Map<K, V>`          | `Map<K, V>`          | `MutableMap<K, V>`              | `(Mutable)Map<K, V>!`              |
| `Map.Entry<K, V>`    | `Map.Entry<K, V>`    | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |
{:.zebra}

Java数组的映射在这里提到过 [below](java-interop.html#java-arrays)：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}

## Kotlin中的Java泛型

Kotlin的泛型和Java的有些不同（详见 [Generics](generics.html)）。当引入java类型的时候，我们作如下转换：

* Java的通配符转换成类型投射
  * `Foo<? extends Bar>` 转换成 `Foo<out Bar!>!`
  * `Foo<? super Bar>` 转换成 `Foo<in Bar!>!`

* Java的原始类型转换成星号投射
  * `List` 转换成 `List<*>!`, 也就是 `List<out Any?>!`

和Java一样，Kotlin在运行时不保留泛型，即对象不知道传递到他们构造器中的那些参数的的实际类型。
也就是，`ArrayList<Integer>()` 和 `ArrayList<Character>()` 是区分不出来的。
这意味着，不可能用 *is*{: .keyword }-来检测泛型。
Kotlin只允许用*is*{: .keyword }-来检测星号投射的泛型类型:

``` kotlin
if (a is List<Int>) // 错误: 不能检测是否是一个Int的List
// but
if (a is List<*>) // 可以：不保证list里面的内容类型
```

### Java数组

和Java不同，Kotlin里的数组不是协变的。Kotlin不允许我们把`Array<String>` 赋值给 `Array<Any>`，
从而避免了可能的运行时错误。Kotlin也禁止我们把一个子类的数组当做父类的数组传递进Kotlin的方法里。
但是对Java方法，这是允许的（考虑这种形式的平台类型[platform types](#platform-types) `Array<(out) String>!`）。

Java平台上，原生数据类型的数组被用来避免封箱/开箱的操作开销。
由于Kotlin隐藏了这些实现细节，就得有一个变通方法和Java代码交互。
每个原生类型的数组都有一个特有类(specialized class)来处理这种问题(`IntArray`, `DoubleArray`, `CharArray` ...)。
它们不是`Array`类，而是被编译成java的原生数组，来获得最好的性能。

假设有一个Java方法，它接受一个表示索引的int数组作参数

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```

在Kotlin里你可以这样传递一个原生数组:

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndices(array)  // passes int[] to method
```

当编译成jvm字节码的时候，编译器会优化对数组的访问，确保不会产生额外的负担。

``` kotlin
val array = arrayOf(1, 2, 3, 4)
array[x] = array[x] * 2 // 不会生成对get() 和 set()的调用
for (x in array) // 不会创建迭代器
  print(x)
```

即便是用索引遍历数组。

``` kotlin
for (i in array.indices) // 不会创建迭代器
  array[i] += 2
```

最后，*in*{: .keyword }-检测也没有额外负担。

``` kotlin
if (i in array.indices) { // 和 (i >= 0 && i < array.size) 一样
  print(array[i])
}
```

## Java Varargs

Java类也会这样声明方法，表示参数是可变参数。

``` java
public class JavaArrayExample {

    public void removeIndices(int... indices) {
        // code here...
    }
}
```

这种情况，你需要用展开操作符 `*` 来传递 `IntArray`：

``` kotlin
val javaObj = JavaArray()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

目前无法传递 *null*{: .keyword } 给一个变参的方法。

## 操作符

虽然Java不能自定义操作符重载，但Kotlin允许任意
使用方法名合法的方法与标示符进行操作符重载，也可以自定义其它约定（如`invoke()`等）。
但调用Java代码的时候，使用中缀语法(`infix call syntax`)是不被允许的。


## 受检异常

在Kotlin里，所有的异常都是非受检的, 也就是说，编译器不会强制你去捕捉任何异常。
因此，你调用一个声明了异常的java方法的时候，kotlin不会强制你作处理。

``` kotlin
fun render(list: List<*>, to: Appendable) {
  for (item in list)
    to.append(item.toString()) // Java里会让你在这里捕捉IOException
}
```

## 对象方法

当java类型被引入到kotlin里时，所有的`java.lang.Object`类型引用，会被转换成 `Any`。
因为`Any`不是平台独有的，它仅声明了三个成员方法：`toString()`, `hashCode()` 和 `equals()`，
所以为了能用到`java.lang.Object`的其他方法，kotlin采用了[扩展函数](extensions.html)。

### wait()/notify()

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第69条善意的提醒了要用concurrency类而不是`wait()` 和 `notify()`。
因此，`Any`不提供这两个方法。
你一定要用的话，就把它转换成`java.lang.Object`。

```kotlin
(foo as java.lang.Object).wait()
```

### getClass()

获取一个对象的类型信息，我们可以用javaClass这个扩展属性。

``` kotlin
val fooClass = foo.javaClass
```

用javaClass<Foo>()，而不是java里的写法`Foo.class`。


``` kotlin
val fooClass = javaClass<Foo>()
```

### clone()

要重写`clone()`，扩展`kotlin.Cloneable`：

```kotlin

class Example : Cloneable {
  override fun clone(): Any { ... }
}
```

不要忘了 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html),第11条: *谨慎的重写克隆*。

### finalize()

要重载 `finalize()`, 你要做的仅仅是声明它，不需要 *override*{:.keyword} 关键字：

```kotlin
class C {
  protected fun finalize() {
    // 具体逻辑
  }
}
```

根据 Java 的规则， `finalize()`不能为 *private*{: .keyword }。

## 从 Java类的继承

在kotlin里，超类里最多只能有一个java类(java接口数目不限)。这个java类必须放在超类列表的最前面。

### 访问静态成员

java类的静态成员就是它们的 “伴生对象”。我们无法将这样的“伴生对象”当作数值来传递，
但可以显式的访问它们，比如：

``` kotlin
if (Character.isLetter(a)) {
  // ...
}
```

## Java 反射

Java反射可以用在kotlin类上，反之亦然。前面提过，你可以 `instance.javaClass` 或者
`ClassName::class.java` 开始基于 `java.lang.Class` 的java反射操作。

Other supported cases include acquiring a Java getter/setter method or a backing field for a Kotlin property, a `KProperty` for a Java field, a Java method or constructor for a `KFunction` and vice versa.

## SAM(单抽象方法) 转换

就像 Java 8 那样，Kotlin 支持 SAM 转换，这意味着 Kotlin 函数字面量可以被自动的转换成
只有一个非默认方法的 Java 接口的实现，只要这个方法的参数类型
能够跟这个 Kotlin 函数的参数类型匹配的上。

你可以这样创建SAM接口的实例：

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...在方法调用里:

``` kotlin
val executor = ThreadPoolExecutor()
// Java签名: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

如果 Java 类有多个接受函数接口的方法，你可以用一个
适配函数来把闭包转成你需要的 SAM 类型。编译器也会在必要时生成这些适配函数。

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

注意SAM的转换只对接口有效，对抽象类无效，即使它们就只有一个
抽象方法。

还要注意这个特性只针对和 Java 的互操作；因为 Kotlin 有合适的函数类型，把函数自动转换成
Kotlin 接口的实现是没有必要的，也就没有支持了。

## 在Kotlin中使用JNI

如果要声明一个使用本机代码(C 或者 C++)实现的方法，你需要给它加上`external`标识符(等同于Java里的`native`)

``` kotlin
external fun foo(x: Int): Double
```

余下的工作和Java完全一样
