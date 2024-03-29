---
title: vi
weight: 713
menu:
  notes:
    name: vi
    identifier: notes-linux-shell-vi
    parent: notes-linux-shell
    weight: 713
---

<!-- vi -->


{{< note title="命令" >}}

显示行号
```
:set number
```

---

在`vi`中执行shell命令
```
:!ls-l
```

---

将`shell`命令的结果插入到当前行的下一行
```
:r !date  //读取系统时间并插入到当前行的下一行
```

---

将起始行号和结束行号指定的范围中的内容输入到shell命令command处理，并将处理结果替换起始行号和结束行号指定的范围中的内容
```
:62,72 !sort  //将62行到72行的内容进行排序
```
当前光标所在行，除可以指定行号外，也可以用`.`表示
```
:. !tr [a-z] [A-Z]  //将当前行的小写转为大写
```

---

将起始行号和结束行号所指定的范围的内容作为命令command的输入，不会改变当前编辑的文件的内容
```
:62,72 w !sort  //将62行到72行的内容进行排序，但排序的结果并不会直接输出到当前编辑的文件中，而是显示在vim敲命令的区域
```

---

将某一行作为`shell`命令执行
```
:62 w !shell //将会把第62行的内容作为shell命令来执行并显示结果，而且不会改变当前编辑的文件的内容
```

{{< /note >}}

