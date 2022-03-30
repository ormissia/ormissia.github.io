---
title: go mod
weight: 122
menu:
    notes:
        name: go mod
        identifier: notes-go-advance-gomod
        parent: notes-go-advance
        weight: 122
---

<!-- go chan -->

{{< note title="go chan close" >}}

在`go`的`chan`中，`chan`被关闭后，消费者会继续读取`channel`中的消息。直到消息被全部读取之后使用`i, ok := <-ch`得到的`ok`才会变为false

下面是测试代码以及运行时控制台打印结果：

```go
func main() {
	ch := make(chan int, 3)

	go producer(ch)

	for {
		i, ok := <-ch
		fmt.Printf("consume msg: %d\tok: %v\n", i, ok)
		time.Sleep(time.Second * 3)
	}

}

func producer(ch chan int) {
	for i := 0; i < 10; i++ {
		ch <- i
		fmt.Printf("produce msg: %d\n", i)
		time.Sleep(time.Second)
	}
	close(ch)
	fmt.Println("chan closed")
}
```

输出结果

```shell
produce msg: 0
consume msg: 0	ok: true
produce msg: 1
produce msg: 2
consume msg: 1	ok: true
produce msg: 3
produce msg: 4
consume msg: 2	ok: true
produce msg: 5
consume msg: 3	ok: true
produce msg: 6
consume msg: 4	ok: true
produce msg: 7
consume msg: 5	ok: true
produce msg: 8
consume msg: 6	ok: true
produce msg: 9
chan closed
consume msg: 7	ok: true
consume msg: 8	ok: true
consume msg: 9	ok: true
consume msg: 0	ok: false
consume msg: 0	ok: false
```

{{< /note >}}
