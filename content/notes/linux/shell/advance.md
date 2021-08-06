---
title: Advance
weight: 712
menu:
  notes:
    name: Advance
    identifier: notes-linux-shell-advance
    parent: notes-linux-shell
    weight: 712
---

<!-- Advance Command -->


{{< note title="xargs" >}}
`xargs`是给命令传递参数的一个过滤器，可以将管道或标准输入（stdin）数据转换成命令行参数，也能够从文件的输出中读取数据，一般是和管道一起使用。
格式:
```bash
somecommand | xargs [-item] [command]
```
选项：
```bash
-a file 从文件中读入作为 stdin
-e flag ，注意有的时候可能会是-E，flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止。
-p 当每次执行一个argument的时候询问一次用户。
-n num 后面加次数，表示命令在执行的时候一次用的argument的个数，默认是用所有的。
-t 表示先打印命令，然后再执行。
-i 或者是-I，这得看linux支持了，将xargs的每项名称，一般是一行一行赋值给 {}，可以用 {} 代替。
-r no-run-if-empty 当xargs的输入为空的时候则停止xargs，不用再去执行了。
-s num 命令行的最大字符数，指的是 xargs 后面那个命令的最大命令行字符数。
-L num 从标准输入一次读取 num 行送给 command 命令。
-l 同 -L。
-d delim 分隔符，默认的xargs分隔符是回车，argument的分隔符是空格，这里修改的是xargs的分隔符。
-x exit的意思，主要是配合-s使用。。
```
{{< /note >}}