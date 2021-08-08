---
title: Basic
weight: 131
menu:
  notes:
    name: Basic
    identifier: notes-go-algorithm-basic
    parent: notes-go-algorithm
    weight: 131
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

{{< note title="堆" >}}

> 堆的实质是一棵完全二叉树

堆可分为两种类型：
- 大根堆：所有子树的根节点均为最大值
- 小根堆：所有子树的根节点均为最小值

一般情况下堆可以用一个有序数组来存储  
[0...i...n]
`i`节点的左孩子`index`为`2*i+1`，右孩子为`2i+2`，父节点为`(i-1)/2`

也有一种特例是从1开始(位运算比加减法快)  
[01...i...n]
`i`节点的左孩子`index`为`2*i`即`i<<1`，右孩子为`2i+1`即`i<<1|1`，父节点为`i/2`

{{< /note >}}

