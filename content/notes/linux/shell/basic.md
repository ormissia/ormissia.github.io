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



{{< note title="echo" >}}

`-n`：不换行
`-e`：支持扩展属性

```shell
# 红色显示OK
echo -e "\033[31mOK\033[0m"
# 绿色显示OK
echo -e "\033[32mOK\033[0m"
```

{{< /note >}}


{{< note title="tr" >}}

删除多余重复字符串

```shell
# 删除多余的空格
echo "a   b   c" | tr -s " "
# 输出：a b c
```

```shell
# 删除多余的a
echo "aaaaacccdetaaadfa   c" | tr -s "a"
# 输出：acccdetadfa   c
```

{{< /note >}}


{{< note title="cut" >}}

```shell
# 以冒号为分隔，过滤第一列
cut -d: -f1 /etc/passwd
# 输出当前系统下所有用户名
```

{{< /note >}}


{{< note title="date" >}}

查看系统时间

```shell
date
# Tue Oct 12 13:36:24 CST 2021
```

{{< /note >}}


{{< note title="tzselect" >}}

查看时区

```shell
ls -l /etc/localtime
# lrwxrwxrwx. 1 root root 33 Oct 12 11:32 /etc/localtime -> /usr/share/zoneinfo/Asia/Shanghai
```

---

获取TZ时区

```shell
tzselect
```

输出：

```shell
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the time zone using the Posix TZ format.
#? 

# 选择数字，依次选择地区、国家、城市，即可得到对应时区
# Asia/Shanghai
```

---

修改系统时区（所有用户生效）

```shell
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

{{< /note >}}
