---
title: Basic Types
weight: 311
menu:
  notes:
    name: Basic Types
    identifier: notes-scala-basics-types
    parent: notes-scala-basic
    weight: 311
---
<!-- 基础 -->

{{< note title="表达式" >}}

表达式是可计算的语句。
```
println(1) // 1
println(1 + 1) // 2
println("Hello!") // Hello!
println("Hello," + " world!") // Hello, world!
```
常量（Values）

你可以使用val关键字来给表达式的结果命名。
```
val x = 1 + 1
println(x) // 2
```

{{< /note >}}


{{< note title="函数" >}}

函数是带有参数的表达式。
```
(x: Int) => x + 1
```
{{< /note >}}

{{< note title="方法" >}}

方法的表现和行为和函数非常类似，但是它们之间有一些关键的差别。

方法由def关键字定义。def后面跟着一个名字、参数列表、返回类型和方法体。
```
def addThenMultiply(x: Int, y: Int)(multiplier: Int): Int = (x + y) * multiplier
println(addThenMultiply(1, 2)(3)) // 9
```
{{< /note >}}


{{< note title="字符串拼接" >}}
```
val a = 1
val b = 2
val c = s"$a+$b=${a + b}"
```
{{< /note >}}

