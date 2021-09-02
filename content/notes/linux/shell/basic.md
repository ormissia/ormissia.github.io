---
title: Basic
weight: 711
menu:
  notes:
    name: Basic
    identifier: notes-linux-shell-basic
    parent: notes-linux-shell
    weight: 711
---

<!-- Basic Command -->


{{< note title="快捷键" >}}

回到命令行开头--`Home`
```
Ctrl+a
```

回到命令行的尾部--`End`
```
Ctrl+e
```

---

删除光标前边的所有字符
```
Ctrl+u
```

删除光标后边的所有字符
```
Ctrl+k
```

删除光标前的一个单词
```
Ctrl+w
```

---
输入曾经的命令下的某个单词或字母，按照单词的匹配`history`
```
Ctrl+r
```

{{< /note >}}


{{< note title="cat" >}}

在`cat`输出时候显示行数

```shell
cat -n maim.go
```
{{< /note >}}


{{< note title="wc" >}}
统计文件行、单词、字符数量
格式：

```shell
usage: wc [-clmw] [file ...]
```
统计`main.go`的行、单词、字符数量

```shell
wc main.go
```

选项：

```shell
-l 统计行数
-c 统计字符数
-w 统计单词数
-L 统计最长的行的字符数
```
{{< /note >}}


{{< note title="nc" >}}

简单的文件传输工具

接收方
```shell
nc -l [port] > filename
```
发送方
```shell
nc [ip] [port] < filename
```
{{< /note >}}


{{< note title="gzip" >}}

解压`*.gz`的压缩文件

与`*.tar.gz`文件不同，`*.gz`文件需要用`gzip`来解压

```shell
gzip -d filename
```
{{< /note >}}

{{< note title="hostnamectl" >}}

修改`hostname`，重启也生效

```shell
hostnamectl set-hostname CentOS
```

查看`hostname`

```shell
hostname
```

{{< /note >}}
