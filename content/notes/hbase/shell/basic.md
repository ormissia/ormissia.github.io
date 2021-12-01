---
title: Basic
weight: 811
menu:
  notes:
    name: Basic
    identifier: notes-hbase-shell-basic
    parent: notes-hbase-shell
    weight: 811
---

<!-- Basic Command -->


{{< note title="shell" >}}

在hbase的shell中scan时指定列

```shell
scan 'table_name',{STARTROW=>'start_row',ENDROW=>'end_row',LIMIT=>100,COLUMNS=>['info:type']}
```

`COLUMNS=>['info:type']`中参数为数组，可以指定列簇名和列名

{{< /note >}}

