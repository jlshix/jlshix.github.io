---
title: "Kotlin 中特别的关键字与操作符"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - Kotlin
---

## 关键字

### `as?`: 安全类型转换, 返回结果可能为 `null`

```kotlin
val x: String? = y as? String
```

### `by`: 属性委托

```kotlin
val lazyValue: String by lazy {
  println("computed!")
  "Hello"
}

fun main() {
  println(lazyValue)
  println(lazyValue)
}
```

### `get/set` 属性的 getter/setter

属性的完整声明语法为:
```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```
例如:

```kotlin
class A(x: Int) {
     var size: Int = x
     val isEmpty: Boolean
         get() = size == 0
     var counter: Int = 0
         set(value) {
             if (value > 0) field = value
         }
 }
```

### `vararg`: 可变数量的参数

函数的最后一个参数可以设置为 vararg 用于传入多个

```kotlin
fun <T> asList(vararg ts: T): List<T> {
  val rv = ArrayList<T>()
  for (t in ts) {
    rv.add(t)
  }
  return rv
}

fun main() {
  val list1 = asList(1, 2, 3)
  val list2 = asList(-1, 0, *list1, 4)
}
```

## 操作符

### `!!`: 断言非空. 将任何值转换为非空类型, 若该值为空则抛出 NPE

```kotlin
val l = b!!.length
```

### `?.`: 安全调用. 非空则执行, 为空则为 `null`, 广泛用于链式调用

```kotlin
val a = "Kotlin"
val b: String? = null

println(b?.length)
println(a.length)
```

```kotlin
val name = bob?.department?.head?.name
```

### `?:` Elvis 操作符, 左侧为 `null` 时取右侧值

```kotlin
val l: Int = if(b != null) b.length else -1
val l = b?.length ?: -1
```

因为 throw 和 return 在 Kotlin 中都是表达式, 所以也可以在右侧:

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
}
```

## 参见

1. [关键字与操作符](https://www.kotlincn.net/docs/reference/keyword-reference.html)