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


{{< note title="时间的加减法" >}}

```go
	// Add 时间相加
	now := time.Now()
	// ParseDuration parses a duration string.
	// A duration string is a possibly signed sequence of decimal numbers,
	// each with optional fraction and a unit suffix,
	// such as "300ms", "-1.5h" or "2h45m".
	//  Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".
	// 10分钟前
	m, _ := time.ParseDuration("-1m")
	m1 := now.Add(m)
	fmt.Println(m1)

	// 8个小时前
	h, _ := time.ParseDuration("-1h")
	h1 := now.Add(8 * h)
	fmt.Println(h1)

	// 一天前
	d, _ := time.ParseDuration("-24h")
	d1 := now.Add(d)
	fmt.Println(d1)

	// 10分钟后
	mm, _ := time.ParseDuration("1m")
	mm1 := now.Add(mm)
	fmt.Println(mm1)

	// 8小时后
	hh, _ := time.ParseDuration("1h")
	hh1 := now.Add(hh)
	fmt.Println(hh1)

	// 一天后
	dd, _ := time.ParseDuration("24h")
	dd1 := now.Add(dd)
	fmt.Println(dd1)

	// Sub 计算两个时间差
	subM := now.Sub(m1)
	fmt.Println(subM.Minutes(), "分钟")

	sumH := now.Sub(h1)
	fmt.Println(sumH.Hours(), "小时")

	sumD := now.Sub(d1)
	fmt.Printf("%v 天\n", sumD.Hours()/24)
```

{{< /note >}}
