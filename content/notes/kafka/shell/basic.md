---
title: Basic
weight: 911
menu:
  notes:
    name: Basic
    identifier: notes-kafka-shell-basic
    parent: notes-kafka-shell
    weight: 911
---

<!-- Basic Command -->


{{< note title="shell" >}}

查询消费者组

```shell
kafka-consumer-groups.sh --bootstrap-server bigdata7:9092 --list
kafka-consumer-groups.sh --bootstrap-server bigdata7:9092 --describe --group group_name
```

删除消费者组

```shell
kafka-consumer-groups.sh --bootstrap-server bigdata7:9092 --delete --group group_name
```

查看`topic`消息数量

```shell
kafka-run-class.sh kafka.tools.GetOffsetShell --bootstrap-server bigdata7:9092 --topic topic_name --time -1
```

{{< /note >}}

