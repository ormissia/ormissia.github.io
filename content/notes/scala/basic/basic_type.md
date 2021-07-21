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


{{< note title="对象" >}}
约等于static单例对象
```
object TestObj {
  def main(args: Array[String]): Unit = {
    val a = 1
    val b = 2
    val c = s"$a+$b=${a + b}"
    println(c)
  }
}
```
{{< /note >}}


{{< note title="类" >}}

> 可以使用`class`关键字定义一个类，后面跟着它的名字和构造参数。
- 类里，裸露的代码是默认构造中的
- 类名构造器中的参数就是类的成员属性，默认是`val`类型，且是`private`
- 只有在类名构造器中的参数可以设置成`var`，其他方法函数中的参数都是`val`类型的，且不允许设置成`var`类型
```
class Greeter(prefix: String, var suffix: String) {
  var name = "name"

  def greet(name: String): Unit =
    println(prefix + name + suffix)
}
```
{{< /note >}}
