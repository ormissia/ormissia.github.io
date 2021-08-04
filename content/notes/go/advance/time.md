---
title: time
weight: 121
menu:
  notes:
    name: time
    identifier: notes-go-advance-time
    parent: notes-go-advance
    weight: 121
---

<!-- Time Type -->

{{< note title="时间转换" >}}
字符串转时间
```go
time.Parse()
```
时间转字符串
```go
time.Format()
```
时间转时间戳
```go
Time.Unix()
```
时间戳转时间
```go
time.Unix()
```
{{< /note >}}


{{< note title="计时" >}}

朴素方法

```go
	startTime := time.Now()
	//do something
	time.Sleep(time.Second)
	duration := time.Since(startTime)
	fmt.Printf("经过时间：%v\n", duration)
	
//经过时间：1.005046959s
```

---

简洁方法

```go
// TimeCost 耗时统计函数
func TimeCost(start time.Time) {
	duration := time.Since(start)
	fmt.Printf("经过时间：%v\n", duration)
}

	defer TimeCost(time.Now())
	//do something
	time.Sleep(time.Second)
	
//经过时间：1.005054375s
```

---

优雅方法

```go
// TimeCost 耗时统计函数
func TimeCost() func() {
	start := time.Now()
	return func() {
		duration := time.Since(start)
		fmt.Printf("经过时间：%v\n", duration)
	}
}

	defer TimeCost()()
	//do something
	time.Sleep(time.Second)
	
//经过时间：1.005033916s
```

{{< /note >}}

