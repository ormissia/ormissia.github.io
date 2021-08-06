---
title: Basic
weight: 011
menu:
  notes:
    name: Basic
    identifier: notes-algorithm-basics-basic
    parent: notes-algorithm-basics
    weight: 011
---

<!-- Basic Type -->

{{< note title="异或" >}}

- 异或运算法则：无进位相加
- 异或运算性质：
  - 0 ^ N = N
  - N ^ N = 0
  - 满足交换律和结合律

```go
a := 0b1100
b := 0b1001
fmt.Printf("%b",a^b)
//101
```

简单应用：不申请额外内存交换两个变量的值

```go
a := 0b1100
b := 0b1001

a = a ^ b
b = a ^ b //b = (a ^ b) ^ b = a
a = a ^ b //a = (a ^ b) ^ a = b
fmt.Printf("a:%b,b:%b", a, b)
//a:1001,b:1100
```

{{< /note >}}
