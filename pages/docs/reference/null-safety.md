---
type: doc
layout: reference
category: "Syntax"
title: "空安全"
---

# 空安全

## 可空类型与非空类型

Kotlin 的类型系统旨在消除来自代码空引用的危险，也称为[《十亿美元的错误》](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)。

许多编程语言（包括 Java）中最常见的陷阱之一是访问空引用的成员，导致空引用异常。在 Java 中，
这等同于 `NullPointerException` 或简称 `NPE`。

Kotlin 的类型系统旨在从我们的代码中消除 `NullPointerException`。在Kotlin中，出现NPE 的唯一可能的原因可能是

* 显式调用 `throw NullPointerException()`；
* 使用了下文描述的 `!!` 操作符；
* 外部 Java 代码导致的；
* 对于初始化，有一些数据不一致（如一个未初始化的 `this` 用于构造函数的某个地方）。

在 Kotlin 中，类型系统区分一个引用可以容纳 *null*{: .keyword } （可空引用）还是不能容纳（非空引用）。
例如，String 类型的常规变量不能容纳 *null*{: .keyword }：

``` kotlin
var a: String = "abc"
a = null // 编译错误
```

如果要允许为空，我们可以声明一个变量为可空字符串，写作 `String?`：

``` kotlin
var b: String? = "abc"
b = null // ok
```

现在，如果你调用 `a` 的方法或者访问它的属性，它保证不会导致 `NPE`，这样你就可以放心地使用：

``` kotlin
val l = a.length
```

但是如果你想访问 `b` 的同一个属性，那么这是不安全的，并且编译器会报告一个错误：

``` kotlin
val l = b.length // 错误：变量“b”可能为空
```

但是我们还是需要访问该属性，对吧？有几种方式可以做到。

## 在条件中检查 *null*{: .keyword }

首先，你可以显式检查 `b` 是否为 *null*{: .keyword }，并分别处理两种可能：

``` kotlin
val l = if (b != null) b.length else -1
```

编译器会跟踪所执行检查的信息，并允许你在 *if*{: .keyword } 内部调用 `length`。
同时，也支持更复杂（更智能）的条件：

``` kotlin
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```

请注意，这只适用于 `b` 是不可变的情况（即在检查和使用之间没有修改过的局部变量
，或者不可覆盖并且有幕后字段的 *val*{: .keyword } 成员），因为否则可能会发生<!--
-->在检查之后 `b` 又变为 *null*{: .keyword } 的情况。

## 安全的调用

你的第二个选择是安全调用操作符，写作 `?.`：

``` kotlin
b?.length
```
如果 `b` 非空，就返回 `b.length`，否则返回 *null*{: .keyword }，这个表达式的类型是 `Int?`。

安全调用在链式调用中很有用。例如，如果一个员工 Bob 可能会（或者不会）分配给一个部门，
并且可能有另外一个员工是该部门的负责人，那么获取 Bob 所在部门负责人（如果有的话）的名字，我们写作：

``` kotlin
bob?.department?.head?.name
```

如果任意一个属性（环节）为空，这个链式调用就会返回 *null*{: .keyword }。

如果要只对非空值执行某个操作，安全调用操作符可以与 [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html) 一起使用：

``` kotlin
val listWithNulls: List<String?> = listOf("A", null)
for (item in listWithNulls) {
     item?.let { println(it) } // 输出 A 并忽略 null
}
```

## Elvis 操作符

当我们有一个可空的引用 `r` 时，我们可以说“如果 `r` 非空，我使用它；否则使用某个非空的值 `x`”：

``` kotlin
val l: Int = if (b != null) b.length else -1
```

除了完整的 *if*{: .keyword }-表达式，这还可以通过 Elvis 操作符表达，写作 `?:`：

``` kotlin
val l = b?.length ?: -1
```

如果 `?:` 左侧表达式非空，elvis 操作符就返回其左侧表达式，否则返回右侧表达式。
请注意，当且仅当左侧为空时，才会对右侧表达式求值。

请注意，因为 *throw*{: .keyword } 和 *return*{: .keyword } 在 Kotlin 中都是表达式，所以它们也可以用在
 elvis 操作符右侧。这可能会非常方便，例如，检查函数参数：

``` kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ……
}
```

## `!!` 操作符

第三种选择是为 NPE 爱好者准备的。我们可以写 `b!!` ，这会返回一个非空的 `b` 值
（例如：在我们例子中的 `String`）或者如果 `b` 为空，就会抛出一个 `NPE` 异常：

``` kotlin
val l = b!!.length
```

因此，如果你想要一个 NPE，你可以得到它，但是你必须显式要求它，否则它不会不期而至。

## 安全的类型转换

如果对象不是目标类型，那么常规类型转换可能会导致 `ClassCastException`。
另一个选择是使用安全的类型转换，如果尝试转换不成功则返回 *null*{: .keyword }：

``` kotlin
val aInt: Int? = a as? Int
```

## 可空类型的集合

如果你有一个可空类型元素的集合，并且想要过滤非空元素，你可以使用 `filterNotNull` 来实现：

``` kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```
