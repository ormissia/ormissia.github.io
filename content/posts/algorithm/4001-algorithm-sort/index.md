---
title: 排序算法
date: 2021-08-07T12:52:56+08:00
hero: head.svg
menu:
  sidebar:
    name: 排序算法
    identifier: algorithm-sort-all
    parent: algorithm
    weight: 4001
---

## 归并排序

> 思想：整体是递归（当然可以用非递归实现），使左边有序，使右边有序，合并左边右边使整体有序

[具体实现](https://github.com/ormissia/go-algorithm/blob/4f24160d20d8bed12ae4e3cf057dee5129b94554/basic/sort/merge_sort.go#L7)

核心代码：

```go
func merge(arr []interface{}, l, mid, r int, compare Compare) {
	help := make([]interface{}, r-l+1)
	i := 0
	p1 := l
	p2 := mid + 1
	for p1 <= mid && p2 <= r {
		if compare(arr[p1], arr[p2]) {
			help[i] = arr[p1]
			p1++
		} else {
			help[i] = arr[p2]
			p2++
		}
		i++
	}
	//要么p1越界了，要么p2越界了
	for p1 <= mid {
		help[i] = arr[p1]
		i++
		p1++
	}
	for p2 <= r {
		help[i] = arr[p2]
		i++
		p2++
	}
	for j, _ := range help {
		arr[l+j] = help[j]
	}
}
```

### 递归

核心代码

```go
func mergeProcess(arr []interface{}, l, r int, compare Compare) {
	if l == r {
		return
	}
	mid := l + ((r - l) >> 1)
	mergeProcess(arr, l, mid, compare)
	mergeProcess(arr, mid+1, r, compare)
	merge(arr, l, mid, r, compare)
}
```

### 非递归

核心代码：

```go
	n := len(arr)
	mergeSize := 1 //当前有序的左组长度
	for mergeSize < n {
		l := 0

		for l < n {
			m := l + mergeSize - 1
			if m >= n {
				break
			}
			r := m + mergeSize
			if m+mergeSize > n-1 {
				r = n - 1
			}
			merge(arr, l, m, r, s.Compare)
			l = r + 1
		}

		//防止溢出
		if mergeSize > n/2 {
			break
		}
		mergeSize <<= 1
	}
```

## 快速排序

[具体实现](https://github.com/ormissia/go-algorithm/blob/4f24160d20d8bed12ae4e3cf057dee5129b94554/basic/sort/quick_sort.go#L10)

> 思想：给定一个数组`arr`和一个整数`num`，把小于等于`num`的数放在数组左边，大于`num`的数放在数组的右边。

> 额外空间复杂度是O(1)，时间复杂度O(N)

核心代码：

```go
func netherlandsFlag(arr []interface{}, l, r int, isEqual, isSmall Compare) (i, j int) {
	if l > r {
		return -1, -1
	}
	if l == r {
		return l, r
	}
	less := l - 1 //小于arr[R]区	右边界
	more := r     //大于arr[R]区	左边界
	index := l
	for index < more {
		if isEqual(arr[index], arr[r]) {
			index++
		} else if isSmall(arr[index], arr[r]) {
			less++
			arr[index], arr[less] = arr[less], arr[index]
			index++
		} else {
			more--
			arr[index], arr[more] = arr[more], arr[index]
		}
	}
	arr[more], arr[r] = arr[r], arr[more]
	return less + 1, more
}
```

### 递归

核心代码：

```go
func quickProcess(arr []interface{}, l, r int, isEqual, isSmall Compare) {
	if l >= r {
		return
	}

	n := rand.Intn(r-l+1) + l
	arr[n], arr[r] = arr[r], arr[n]
	i, j := netherlandsFlag(arr, l, r, isEqual, isSmall)
	quickProcess(arr, l, i-1, isEqual, isSmall)
	quickProcess(arr, j+1, r, isEqual, isSmall)
}
```

## 堆排序

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

> 堆排序思想：依次弹出

## 桶排序
