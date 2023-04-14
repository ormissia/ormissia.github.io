---
title: Golang中的原子操作
date: 2022-03-30T16:13:12+08:00
hero: head.svg
menu:
  sidebar:
    name: Golang中的原子操作
    identifier: golang-atomic
    parent: go
    weight: 006
---

---

#golang #atomic

---

## 互斥锁跟原子操作的区别

在并发编程里，`Go`语言`sync`包里的同步原语`Mutex`是我们经常用来保证并发安全的，但是他跟`atomic`包在使用目的和底层实现上都不一样：

### 使用目的

互斥锁是用来保护一段逻辑，原子操作用于对一个变量的更新保护。

### 底层实现

`Mutex`由操作系统的调度器实现，而`atomic`包中的原子操作则由底层硬件指令直接提供支持，这些指令在执行的过程中是不允许中断的，因此原子操作可以在`lock-free`的情况下保证并发安全，并且它的性能也能做到随`CPU`个数的增多而线性扩展。


__对于一个变量更新的保护，原子操作通常会更有效率，并且更能利用计算机多核的优势。__

## 性能测试对比

### 互斥锁性能测试

> 使用`sync`包下面互斥锁的多线程加法操作

```go
func syncAdd(param int64) int64 {
	var wg sync.WaitGroup
	lock := sync.Mutex{}
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			for i := 0; i < 1000000; i++ {
				lock.Lock()
				param++
				lock.Unlock()
			}
			wg.Done()
		}()
	}
	wg.Wait()
	return param
}
```

Benchmark测试方法

```go
func BenchmarkSync(b *testing.B) {
	for i := 0; i < b.N; i++ {
		flag := int64(0)
		res := syncAdd(flag)
		if res != 10000000 {
			b.Errorf("calculate result err: %d\n", res)
		}
	}
}
```

测试结果：

> 根据运行环境和硬件性能会有所不同，这里是在相同环境下的对比

- 第一次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkSync
BenchmarkSync-8   	       2	 862741542 ns/op
PASS
```

- 第二次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkSync
BenchmarkSync-8   	       2	 875432729 ns/op
PASS
```

- 第三次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkSync
BenchmarkSync-8   	       2	 836373292 ns/op
PASS
```

三次取平均：`(862741542 + 875432729 + 836373292) / 3 = 858182521 ns/op`

### 原子操作性能测试

> 使用`atomic`包下面原子操作的多线程加法操作

```go
func atomicAdd(param int64) int64 {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			for i := 0; i < 1000000; i++ {
				atomic.AddInt64(&param, 1)
			}
			wg.Done()
		}()
	}
	wg.Wait()
	return param
}
```

Benchmark测试方法

```go
func BenchmarkAtomic(b *testing.B) {
	for i := 0; i < b.N; i++ {
		flag := int64(0)
		res := atomicAdd(flag)
		if res != 10000000 {
			b.Errorf("calculate result err: %d\n", res)
		}
	}
}
```

测试结果：

> 根据运行环境和硬件性能会有所不同，这里是在相同环境下的对比

- 第一次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkAtomic
BenchmarkAtomic-8   	       4	 359013958 ns/op
PASS
```

- 第二次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkAtomic
BenchmarkAtomic-8   	       3	 359734514 ns/op
PASS
```

- 第三次

```shell
goos: darwin
goarch: arm64
pkg: wire/atomic_test
BenchmarkAtomic
BenchmarkAtomic-8   	       4	 359007542 ns/op
PASS
```

> 三次取平均：`(359013958 + 359734514 + 359007542) / 3 = 359252004 ns/op`

### 测试结果对比

根据测试结果数据使用互斥锁做累加每次循环耗时`858182521 ns`，而使用原子操作做累加每次耗时`359252004 ns`。

这也印证了之前说过的：`互斥锁适用于来保护一段逻辑，原子操作适用于于对一个变量的更新保护。`

### 原理浅析

参考： [互斥锁跟原子操作的区别-底层实现](https://ormissia.github.io/posts/knowledge/2001-go/006-atomic/#%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0)

## 参考链接

- [Golang五种原子性操作的用法详解](https://zhuanlan.zhihu.com/p/412666957)
