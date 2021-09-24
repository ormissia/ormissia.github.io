---
title: Basic
weight: 411
menu:
  notes:
    name: Basic
    identifier: notes-python-basics-types
    parent: notes-python-basic
    weight: 411
---
<!-- 基础 -->


{{< note title="函数" >}}

在 python 中，类型属于对象，变量是没有类型的：

```python
a=[1,2,3]
a="ormissia"
```

以上代码中，`[1,2,3]`是`List`类型，`"ormissia"`是`String`类型，而变量`a`是没有类型，他仅仅是一个对象的引用（一个指针），可以是指向`List`类型对象，也可以是指向`String`类型对象。

## 可更改(mutable)与不可更改(immutable)对象

在python中，`strings`，`tuples`和`numbers`是不可更改的对象，而`list`，`dict`等则是可以修改的对象。

- 不可变类型：变量赋值`a=5`后再赋值`a=10`，这里实际是新生成一个`int`值对象`10`，再让`a`指向它，而`5`被丢弃，不是改变`a`的值，相当于新生成了`a`。

- 可变类型：变量赋值`la = [1,2,3,4]`后再赋值`la[2] = 5`则是将`list la`的第三个元素值更改，本身`la`没有动，只是其内部的一部分值被修改了。

## python函数的参数传递：

- 不可变类型：类似`C++`的值传递，如整数、字符串、元组。如`fun(a)`，传递的只是`a`的值，没有影响`a`对象本身。如果在`fun(a)`内部修改`a`的值，则是新生成一个`a`的对象。

- 可变类型：类似`C++`的引用传递，如列表，字典。如`fun(la)`，则是将`la`真正的传过去，修改后`fun`外部的`la`也会受影响

python中一切都是对象，严格意义我们不能说值传递还是引用传递，我们应该说传不可变对象和传可变对象。


{{< /note >}}
