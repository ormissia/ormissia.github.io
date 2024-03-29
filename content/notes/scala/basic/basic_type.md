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

函数参数为`val`类型，且可以给出默认值
```
def test(a: Int, b: Int = 1, c: Int = 2): Unit = {
  println(s"$a $b $c")
}
  
test(1, 2)      //1 2 2
test(1, c = 4)  //1 1 4
```
{{< /note >}}

{{< note title="匿名函数" >}}

函数是带有参数的表达式。
```
(x: Int) => x + 1
```

{{< /note >}}


{{< note title="方法" >}}

方法的表现和行为和函数非常类似，但是它们之间有一些关键的差别。

方法由`def`关键字定义。`def`后面跟着一个名字、参数列表、返回类型和方法体。
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
约等于`static`单例对象
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

可以使用`class`关键字定义一个类，后面跟着它的名字和构造参数。
- 类里裸露的代码是默认构造中的
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


{{< note title="循环" >}}
`scala`中嵌套`for`循环可以写到一起，循环上可以加守卫（条件）。
循环结果可以通过`yield`收集到一个集合中
```go
// val value = for (i <- 1 to 9; j <- 1 to i) yield {
val value = for (i <- 1 to 9; j <- 1 to 10 if (j <= i)) yield {
  i * j
}
for (i <- value) {
  println(i)
}
```
{{< /note >}}


{{< note title="偏应用函数" >}}

类似于重新封装一下函数
```
def log(date: Date, logType: String, msg: String): Unit = {
  println(s"$date\t$logType\t$msg")
}

val info = log(_, "info", _)
info(new Date, "this is a info msg")  //Thu Jul 22 23:14:04 CST 2021	info	this is a info msg
```

{{< /note >}}


{{< note title="可变长度参数以及foreach" >}}

```
def foreachTest(a: Int*): Unit = {
  //for (i <- a) {
  //  print(i)
  //}

  //a.foreach((x: Int) => {
  //  print(x)
  //})

  //a.foreach(print(_))
  a.foreach(print)
}

foreachTest(1, 2, 3, 4, 5)  //12345
```

{{< /note >}}


{{< note title="高阶函数" >}}

函数作为参数
```
def computer(a: Int, b: Int, f: (Int, Int) => Int): Unit = {
  val res = f(a, b)
  println(res)
}

computer(1, 2, (x: Int, y: Int) => {x + y}) //3
computer(1, 2, _ + _)                       //3
```

---

函数作为返回值

```
def factory(i: String): (Int, Int) => Int = {
  def plus(x: Int, y: Int): Int = {
    x + y
  }

  if (i.equals("+")) {
    plus
  } else {
    _ * _
  }
}

val plus = factory("+")
computer(1,2,plus)  //3
```
{{< /note >}}


{{< note title="柯里化" >}}

多个参数列表
```
def testFunc(a:Int*)(b:Int*)(c:String*): Unit ={
  a.foreach(print)
  b.foreach(print)
  c.foreach(print)
}

testFunc(1,2,3)(2,3,4)("3","4","5") //123234345
```

{{< /note >}}


{{< note title="数组" >}}

数组
`scala`中泛型是`[]`，数组用`()`

`val`约等于`final`，不可变描述的是`val`指定的引用（字面值、地址）
```
val arr1 = Array[Int](1, 2, 3)
arr1(1) = 99
println(arr1(1))  //99


```

---

遍历

```

for (elem <- arr1) {}
//foreach需要函数接收元素
arr1.foreach(println)
```

{{< /note >}}


{{< note title="链表" >}}

`scala`中`collections`中有两个包：`immutable,mutable`，默认是不可变的`immutable`

```
val list1 = List(1, 2, 3, 4)

//++ += ++: :++
val list2 = new ListBuffer[Int]
list2.+=(1)
list2.+=(2)
list2.+=(3)
```

---

```
val list1 = List(1, 2, 3, 4)
val list2 = list1.map(_ * 2)
list2.foreach(print) //2468
```

{{< /note >}}


{{< note title="Set" >}}

`Set`

不可变的

```
val set1 = Set(1, 2, 3, 4, 1, 2)  //1 2 3 4
```

---

可变的

```
val set2 = mutable.Set(1, 2, 3, 4, 1, 2)
set2.add(1)
set2.add(5) //1 2 3 4 5
```


{{< /note >}}


{{< note title="Map" >}}

`Map`

```
val map1 = Map(("a", 1), "b" -> 2, ("c", 3), ("a", 4))

map1.foreach(print) //(a,4)(b,2)(c,3)
println(map1.get("a")) //Some(4)
println(map1.get("d")) //None
println(map1.getOrElse("a", "test")) //4
println(map1.getOrElse("d", "test")) //test

val keys = map1.keys
keys.foreach(println)
```

---

遍历

```
for (m <- map1) {
  print(s"$m")
}
for (k <- keys) {
  print(s"($k,${map1(k)})")
}
```

---

可变的

```
val map2 = mutable.Map(("a", 1), "b" -> 2, ("c", 3), ("a", 4))
map2.put("a", 5)
```

{{< /note >}}


{{< note title="案例类" >}}
{{< /note >}}


{{< note title="模式匹配" >}}
{{< /note >}}


{{< note title="特质" >}}
{{< /note >}}


{{< note title="偏函数" >}}
{{< /note >}}


{{< note title="隐式转换" >}}
{{< /note >}}
