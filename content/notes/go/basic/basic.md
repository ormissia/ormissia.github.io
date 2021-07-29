---
title: Basic
weight: 111
menu:
  notes:
    name: Basic
    identifier: notes-go-basics-basic
    parent: notes-go-basics
    weight: 111
---

<!-- Basic Type -->

{{< note title="基础类型内存宽度以及表示范围" >}}

`bool`
```
1Byte true/false
```
`uint8`
```
1Byte 0-255
```
`uint16`
```
2Byte 0-65535
```
`uint32`
```
4Byte 0-4294967295
```
`uint64`
```
8Byte 0-18446744073709551615
```

`int8`
```
1Byte -128-127
```
`int16`
```
2Byte -32768-32767
```
`int32`
```
4Byte -2147483648-2147483647
```
`int64`
```
6Byte -9223372036854775808-9223372036854775807
```
`byte`
```
1Byte 类似 uint8
```
`rune`
```
4Byte 类似 int32
```
`uint`
```
4Byte / 8Byte 32 或 64 位
```
`int`
```
4Byte / 8Byte 与 uint 一样大小
```
`float32`
```
4Byte
```
`float64`
```
8Byte
```
`string`
```
1Byte （英文） / 2Byte-4Byte（中文，取决于字符编码类型）
```

{{< /note >}}


