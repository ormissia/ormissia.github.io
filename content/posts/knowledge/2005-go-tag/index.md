---
title: Golang struct tag浅析与自定义tag实践
date: 2021-08-13T16:55:47+08:00
hero: head.svg
menu:
  sidebar:
    name: Golang struct tag
    identifier: knowledge-go-struct-tag
    parent: knowledge
    weight: 2005
---

> StructTag是写在结构体字段类型后面反引号中的内容，用来标记结构体中各字段的属性。

> 源码中对`tag`的解释：
> > By convention, tag strings are a concatenation of optionally space-separated key:"value" pairs.
> > Each key is a non-empty string consisting of non-control characters other than space (U+0020 ' '), quote (U+0022 '"'),
> > and colon (U+003A ':').  Each value is quoted using U+0022 '"' characters and Go string literal syntax.

## 简单应用

最常见的，比如`json`的`tag`应用：

`json`序列化和反序列化时候使用的`key`都是在`struct`字段上定义的

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	ID       int    `json:"id"`
	Username string `json:"username"`
	Age      int    `json:"age"`
	Email    string `json:"email"`
}

func main() {
	u := User{
		ID:       1,
		Username: "ormissia",
		Age:      90,
		Email:    "email@example.com",
	}

	userJson, _ := json.Marshal(u)
	fmt.Println(string(userJson))

	u2Str := `{"id":2,"username":"ormissia","age":900,"email":"ormissia@example.com"}`
	u2 := new(User)
	_ = json.Unmarshal([]byte(u2Str), u2)
	fmt.Printf("%+v",u2)
}
```

输出：

```bash
{"id":1,"username":"ormissia","age":90,"email":"email@example.com"}
&{ID:2 Username:ormissia Age:900 Email:ormissia@example.com}
```

## tag解析原理

### 通过反射拿到struct tag

示例：

```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	ID       int    `json:"id" myTag:"ID"`
	Username string `json:"username" myTag:"USERNAME"`
	Age      int    `json:"age" myTag:"AGE"`
	Email    string `json:"email" myTag:"EMAIL"`
}

func main() {
	u := User{1, "ormissia", 90, "email@example.com"}

	userTyp := reflect.TypeOf(u)
	fieldTag := userTyp.Field(0).Tag
	fmt.Printf("user field 0 id tag: %s\n",fieldTag)
	value, ok1 := fieldTag.Lookup("myTag")
	fmt.Println(value, ok1)
	value1, ok2 := fieldTag.Lookup("other")
	fmt.Println(value1, ok2)
}
```

输出：

```go
user field 0 id tag: json:"id" myTag:"ID"
ID true
 false
```

### 获取tag全部的值

```go
reflect.TypeOf(u).Field(0).Tag
```

通过`Tag`即可获取`struct`定义时候对应字段后面反引号\`\`中全部的值

`Tag`是通过反射获取到的具体字段[StructField](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1090)
中的属性，类型为自定义`string`类型：[StructTag](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1113)

```go
type StructField struct {
	// Name is the field name.
	Name string
	// PkgPath is the package path that qualifies a lower case (unexported)
	// field name. It is empty for upper case (exported) field names.
	// See https://golang.org/ref/spec#Uniqueness_of_identifiers
	PkgPath string

	Type      Type      // field type
	Tag       StructTag // field tag string
	Offset    uintptr   // offset within struct, in bytes
	Index     []int     // index sequence for Type.FieldByIndex
	Anonymous bool      // is an embedded field
}
```

```go
// A StructTag is the tag string in a struct field.
//
// By convention, tag strings are a concatenation of
// optionally space-separated key:"value" pairs.
// Each key is a non-empty string consisting of non-control
// characters other than space (U+0020 ' '), quote (U+0022 '"'),
// and colon (U+003A ':').  Each value is quoted using U+0022 '"'
// characters and Go string literal syntax.
type StructTag string
```

### 通过tag的key获取value


而[StructTag](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1113)
有两个通过`key`获取`value`的方法：
[Get](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1120)
和[Lookup](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1131) 。

[Get](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1120)
是对[Lookup](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1131)
的一个封装。

[Lookup](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1131)
可以返回当前查询的`key`是否存在。

```go
func (tag StructTag) Get(key string) string {
	v, _ := tag.Lookup(key)
	return v
}
```

```go
func (tag StructTag) Lookup(key string) (value string, ok bool) {
	for tag != "" {
		// Skip leading space.
		i := 0
		for i < len(tag) && tag[i] == ' ' {
			i++
		}
		tag = tag[i:]
		if tag == "" {
			break
		}

		i = 0
		for i < len(tag) && tag[i] > ' ' && tag[i] != ':' && tag[i] != '"' && tag[i] != 0x7f {
			i++
		}
		if i == 0 || i+1 >= len(tag) || tag[i] != ':' || tag[i+1] != '"' {
			break
		}
		name := string(tag[:i])
		tag = tag[i+1:]

        //...
        //...
}
```

通过源码，我们可以看出
[Lookup](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1131)
实际上是对`tag`反引号中整个内容进行查找，通过空格、冒号以及双引号对`tag`的值进行分割，最后返回。

## 使用自定义tag实践

我们可以一个`struct`参数校验器：[go-opv](https://github.com/ormissia/go-opv) 来简单体验一下自定义`tag`的使用。

### go-opv的简单使用示例

> [go-opv](https://github.com/ormissia/go-opv) 的简介可以参考我的
> [另一篇文章](https://ormissia.github.io/posts/knowledge/2002-go-param-verify/) 。
> 只不过，当时还没有添加这个通过`struct`自定义`tag`的校验方式，基础功能与现在基本一致。

仓库[go-opv](https://github.com/ormissia/go-opv)

以下是一个简单的使用Demo

在这个示例中，我们指定了`struct`的`tag`为`go-opv:"ge:0,le:20"`，在参数检验过程中，我们会解析这个tag，并从中获取定义的规则。

```go
package main

import (
	"log"

	go_opv "github.com/ormissia/go-opv"
)

type User struct {
	Name string `go-opv:"ge:0,le:20"`  //Name >=0 && Name <=20
	Age  int    `go-opv:"ge:0,lt:100"` //Age >= 0 && Age < 100
}

func init() {
	//使用默认配置：struct tag名字为"go-opv"，规则与限定值的分隔符为":"
	myVerifier = go_opv.NewVerifier()
	//初始化一个验证规则：Age字段大于等于0，小于200
	userRequestRules = go_opv.Rules{
		"Age": []string{myVerifier.Ge("0"), myVerifier.Lt("200")},
	}
}

var myVerifier go_opv.Verifier
var userRequestRules go_opv.Rules

func main() {
	// ShouldBind(&user) in Gin framework or other generated object
	user := User{
		Name: "ormissia",
		Age:  190,
	}

	//两种验证方式混合,函数参数中传入自定义规则时候会覆盖struct tag上定义的规则
	//根据自定义规则Age >= 0 && Age < 200，Age的值为190，符合规则，验证通过
	if err := myVerifier.Verify(user, userRequestRules); err != nil {
		log.Println(err)
	} else {
		log.Println("pass")
	}

	//只用struct的tag验证
	//根据tag上定义的规则Age >= 0 && Age < 100，Age的值为190，不符合规则，验证不通过
	if err := myVerifier.Verify(user); err != nil {
		log.Println(err)
	} else {
		log.Println("pass")
	}
}
```

### go-opv的简单分析

我们先判断了是否传入了自定义的校验规则（即为自定义规则会覆盖`struct`上`tag`定义的规则），如果没有，就去通过反射获取`struct`上`tag`定义的规则。
然后生成相对应的规则，继续执行后面的校验逻辑。

```go
if len(conditions[tagVal.Name]) == 0 {
	//没有自定义使用tag
	//`go-opv:"ne:0,eq:10"`
	//conditionsStr = "ne:0,eq:10"
	if conditionsStr, ok := tagVal.Tag.Lookup(verifier.tagPrefix); ok && conditionsStr != "" {
		conditionStrs := strings.Split(conditionsStr, ",")
		conditions[tagVal.Name] = conditionStrs
	} else {
		//如果tag也没有定义则去校验下一个字段
		continue
	}
}
```

## 小结

- 通过使用[encoding/json](https://github.com/golang/go/tree/dev.boringcrypto.go1.17/src/encoding/json)
包中的[json.Marshal()](https://github.com/golang/go/blob/ef7be8869c2921cfb6fbcb9c2fa12d7dd8933410/src/encoding/json/encode.go#L158)
和[json.Unmarshal()](https://github.com/golang/go/blob/ef7be8869c2921cfb6fbcb9c2fa12d7dd8933410/src/encoding/json/decode.go#L96)
，我们了解了Golang中`struct tag`的基本概念及用途。
- 通过对[StructField](https://github.com/golang/go/blob/fa6aa872225f8d33a90d936e7a81b64d2cea68e1/src/reflect/type.go#L1090)
的分析，我们明白了`struct tag`工作的基本原理
- 而通过[go-opv](https://github.com/ormissia/go-opv) 的分析，我们了解了自定义`tag`的基本使用方法。

## 参考

- [go1.16.7 type.go 源码](https://github.com/golang/go/blob/go1.16.7/src/reflect/type.go)
