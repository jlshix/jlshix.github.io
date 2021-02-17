---
title: "Kotlin 作用域函数"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - Kotlin
---

## 概述

Kotlin 提供了五个作用域函数, 在对象上调用这些函数时传入一个 lambda 表达式作为一个临时作用域.
在此作用域内可以不适用对象的名称进行使用.

这五个作用域函数是: `let, run, with, apply, also`. 功能都是在一个对象上执行一个代码块.
区别在于对象的引用和返回结果.

可以根据如下脑图进行决策:

![五个作用域函数的选择](/img/in-post/2022-02-17-01.jpg)

## 使用示例

### let

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

class LetTest {
    private val numbers = mutableListOf("one", "two", "three", "four", "five")

    @Test
    fun `let it returns lambda result`() {
        val rv = numbers.map { it.length }.filter { it > 3 }.let {
            println(it)
            it
        }
        assertEquals(listOf(5, 4, 4), rv)
    }

    @Test
    fun `let method reference`() {
        val rv = numbers.map { it.length }.filter { it > 3 }.let(::println)
        assertEquals(Unit, rv)
    }

    @Test
    fun `let safe call`() {
        val str: String? = "Hello"
        val length = str?.let {
            println("let() called on $it")
            it.length
        }
        assertEquals(5, length)
    }

    @Test
    fun `let lambda argument name`() {
        val modifiedFirstItem = numbers.first().let { firstItem ->
            println("first item of list is $firstItem")
            if (firstItem.length > 5) firstItem else "!$firstItem!"
        }.uppercase()
        assertEquals("!ONE!", modifiedFirstItem)
    }
}
```

### run

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

class MultiPortService(var url: String, var port: Int) {
    fun prepareRequest(): String = "Default request"
    fun query(request: String): String = "Result for query '$request'"
}

class RunTest {
    private val service = MultiPortService("https://example.com", 80)

    @Test
    fun `run this returns lambda result`() {
        val result = service.run {
            port = 8080
            query(prepareRequest() + " to port $port")
        }
        assertEquals("Result for query 'Default request to port 8080'", result)
        assertEquals(8080, service.port)
    }

    @Test
    fun `same code written with let()`() {
        val result = service.let {
            it.port = 8081
            it.query(it.prepareRequest() + " to port ${it.port}")
        }
        assertEquals("Result for query 'Default request to port 8081'", result)
        assertEquals(8081, service.port)
    }

    @Test
    fun `run as non-extension function`() {
        val result = run {
            println("inside non-extension function run()")
            service.port = 8082
            service.query(service.prepareRequest() + " to port ${service.port}")
        }
        assertEquals("Result for query 'Default request to port 8082'", result)
        assertEquals(8082, service.port)
    }
}
```

### with

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

class WithTest {
    private val numbers = listOf("one", "two", "three")

    @Test
    fun `with is not an ext function`() {
        with(numbers) {
            println("'with' is called with argument $this")
            println("it contains $size elements")
            assertEquals(3, size)
            assertEquals(3, this.size)
            assertEquals("three", last())
        }
    }

    @Test
    fun `with returns lambda result`() {
        val firstAndLast = with(numbers) {
            "first: ${first()}; last: ${last()}"
        }
        assertEquals("first: one; last: three", firstAndLast)
    }

}
```

### apply

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

data class Person(
    var name: String,
    var age: Int = 0,
    var city: String = "",
)

class ApplyTest {
    @Test
    fun `apply this returns this`() {
        val adam = Person("Adam").apply {
            age = 32
            city = "London"
        }
        assertEquals(32, adam.age)
        assertEquals("London", adam.city)
    }
}
```

### also

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

class AlsoTest {
    @Test
    fun `also it returns this`() {
        val numbers = mutableListOf("one", "two", "three")
        numbers.also { println("before added: $numbers") }.add("four")
        assertEquals(mutableListOf("one", "two", "three", "four"), numbers)
    }
}
```

## takeIf 与 takeUnless

```kotlin
package com.jlshix.kt.learning.scopeFunctions

import kotlin.test.Test
import kotlin.test.assertEquals

class TakeTest {
    @Test
    fun `takeIf like filter`() {
        val number = 42
        val result = number.takeIf { it % 2 == 0 }?.times(2)?.plus(2)
        assertEquals(86, result)
    }

    @Test
    fun `take null in chain`() {
        val number = 43
        val result = number.takeIf { it % 2 == 0 }?.times(2)?.plus(2)
        assertEquals(null, result)
    }

    @Test
    fun `takeUnless like filterNot`() {
        val number = 42
        val result = number.takeUnless { it % 2 != 0 }?.times(2)?.plus(2)
        assertEquals(86, result)
    }
}
```

## 参见

1. [Scope functions](https://kotlinlang.org/docs/scope-functions.html)