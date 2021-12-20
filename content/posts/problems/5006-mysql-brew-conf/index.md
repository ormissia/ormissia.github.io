---
title: 修改Mac上brew安装的MySQL配置
date: 2021-12-20T09:18:43+08:00
hero: head.svg
menu:
  sidebar:
    name: 修改Mac上brew安装的MySQL配置
    identifier: problems-docker-image-source
    parent: problems
    weight: 5006
---

> 一般MySQL 8.x安装完在`select`语句中使用`group by`时会报错，需要在`my.cnf`中配置设置`sql_model`参数。在Linux中，这个文件通常位于`/etc`目录下，而在Mac上，却不在这里。

在Mac本地安装的测试用的MySQL数据库，安装完成之后需要进行如下设置

#### 设置sql_model

>  关闭ONLY_FULL_GROUP_BY模式

在sql命令行中查询`sql_mode`配置

```sql
select @@sql_mode;
```

```sql
mysql> select @@sql_mode;
+-----------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                            |
+-----------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION |
+-----------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

去掉第一项后得到：

```sql
STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

使用`mysql --help`命令获取`my.cnf`配置文件所在位置

```shell
# ...
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /opt/homebrew/etc/my.cnf ~/.my.cnf
The following groups are read: mysql client
# ...
```

我安装的MySQL在`/opt/homebrew/etc/my.cnf`目录下，添加一行：

```
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

重启MySQL，问题解决

```shell
mysql.server restart
```
