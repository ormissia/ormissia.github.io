# Linux

#操作系统 #linux 

## 进程管理

### 查看进程

#### ps

查看某个时间点的进程信息。

示例：查看自己的进程

```shell
ps -l
```

示例：查看系统所有进程

```shell
ps aux
```

示例：查看特定的进程

```shell
ps aux | grep threadx
```

#### pstree

查看进程树。

示例：查看所有进程树

```shell
pstree -A
```

#### top

实时显示进程信息。

示例：两秒钟刷新一次

```shell
top -d 2
```

#### netstat

查看占用端口的进程

示例：查看特定端口的进程

```shell
netstat -anp | grep port
```

### 进程状态

| 状态 | 说明                                                                                                                                 |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------ |
| R    | running or runnable (on run queue)<br>正在执行或者可执行，此时进程位于执行队列中。                                                   |
| D    | uninterruptible sleep (usually I/O)<br>不可中断阻塞，通常为 IO 阻塞。                                                                |
| S    | interruptible sleep (waiting for an event to complete) <br> 可中断阻塞，此时进程正在等待某个事件完成。                               |
| Z    | zombie (terminated but not reaped by its parent)<br>僵死，进程已经终止但是尚未被其父进程获取信息。                                   |
| T    | stopped (either by a job control signal or because it is being traced) <br> 结束，进程既可以被作业控制信号结束，也可能是正在被追踪。 |



### #孤儿进程

一个父进程退出，而它的一个或多个子进程还在运行，那么这些子进程将成为孤儿进程。

孤儿进程将被 init 进程（进程号为 1）所收养，并由 init 进程对它们完成状态收集工作。

由于孤儿进程会被 init 进程收养，所以孤儿进程不会对系统造成危害。

### #僵尸进程

一个子进程的进程描述符在子进程退出时不会释放，只有当父进程通过 `wait()` 或 `waitpid () ` 获取了子进程信息后才会释放。如果子进程退出，而父进程并没有调用 ` wait () ` 或 ` waitpid () `，那么子进程的进程描述符仍然保存在系统中，这种进程称之为僵尸进程。

僵尸进程通过 ps 命令显示出来的状态为 Z（zombie）。

系统所能使用的进程号是有限的，如果产生大量僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程。

要消灭系统中大量的僵尸进程，只需要将其父进程杀死，此时僵尸进程就会变成孤儿进程，从而被 init 进程所收养，这样 init 进程就会释放所有的僵尸进程所占有的资源，从而结束僵尸进程。


## 查看连接

```shell
netstat -natp
```