---
title: Memory
weight: 721
menu:
  notes:
    name: Memory
    identifier: notes-linux-system-memory
    parent: notes-linux-system
    weight: 721
---

<!-- Memory -->


{{< note title="内存" >}}

一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

- VSS - Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
- RSS - Resident Set Size 实际使用物理内存（包含共享库占用的内存）
- PSS - Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）
- USS - Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）

---

RSS / VSZ

- RSS是`Resident Set Size`（常驻内存大小）的缩写，用于表示进程使用了多少内存（RAM中的物理内存），
  RSS不包含已经被换出的内存。RSS包含了它所链接的动态库并且被加载到物理内存中的内存。RSS还包含栈内存和堆内存。
- VSZ是`Virtual Memory Size`（虚拟内存大小）的缩写。它包含了进程所能访问的所有内存，包含了被换出的内存，
  被分配但是还没有被使用的内存，以及动态库中的内存。

假设进程A的二进制文件是500K，并且链接了一个2500K的动态库，堆和栈共使用了200K，其中100K在内存中（剩下的被换出或者不再被使用），
一共加载了动态库中的1000K内容以及二进制文件中的400K内容至内存中，那么：

```shell
RSS: 400K + 1000K + 100K = 1500K
VSZ: 500K + 2500K + 200K = 3200K
```

由于部分内存是共享的，被多个进程使用，所以如果将所有进程的RSS值加起来可能会大于系统的内存总量。

申请过的内存如果程序没有实际使用，则可能不显示在RSS里。比如说一个程序，预先申请了一大批内存，
过了一段时间才使用，你会发现RSS会增长而VSZ保持不变。

还有一个概念是PSS，它是`proportional set size`（proportional是成比例的意思）的缩写。
这是一种新的度量方式。它将动态库所使用的内存按比例划分。比如我们前面例子中的动态库如果是被两个进程使用，那么：

```shell
PSS: 400K + (1000K/2) + 100K = 400K + 500K + 100K = 1000K
```

一个进程中的多个线程共享同样的地址空间。所以一个进程中的多个线程的RSS，VSZ，PSS是完全相同的。linux下可以使用ps或者top命令查看这些信息。

[英文原文](https://stackoverflow.com/a/21049737)

{{< /note >}}
