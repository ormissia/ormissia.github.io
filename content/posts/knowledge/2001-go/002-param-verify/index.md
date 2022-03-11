---
title: Golang 实体参数校验
date: 2021-07-27T15:08:44+08:00
hero: head.svg
menu:
  sidebar:
    name: 实体参数校验
    identifier: go-param-verify
    parent: go
    weight: 002
---

---

![Golang](https://img.shields.io/badge/-Golang-blue)
![go-opv](https://img.shields.io/badge/-go--opv-lightgrey)
![反射](https://img.shields.io/badge/-%E5%8F%8D%E5%B0%84-orange)
![reflect](https://img.shields.io/badge/-reflect-red)

---

> A："请用一句话让别人知道你写过Golang。"  
> B："if err!= nil ..."

## 起因

只要是接触过Golang的人，无不为其`if err != nil`的语法感到惊奇，或是大加赞赏，或是狠狠痛批。作为使用者，不管喜欢也好，反对也罢，
目前还是要接受这种错误处理模式。  

而最令人头痛的就是请求参数中各种值的校验。比如`Get`请求中接收分页参数时，需要将`string`格式的参数转换成`int`类型，再如时间类型的参数 转换，
诸如此类，等等等等。好家伙，一个接口写完`if err != nil`的判断占了一多半的行数，看着实在不爽。  

下面就是一个典型的例子，而且这个接口参数还不是特别多

```go
func Export(c *gin.Context) {
    //删除开头
    //...
	var param map[string]string
	err := c.ShouldBindJSON(&param)
	if err != nil {
		ErrRsponse(c,errCode)
		return
	}
	var vId, userId, userName, format string
	if v, ok := param["vId"]; ok {
		vId = v
	} else {
		ErrRsponse(c,errCode)
		return
	}

	if len(vId) == 0 {
		ErrRsponse(c,errCode)
		return
	}

	if v, ok := param["userId"]; ok {
		userId = v
	} else {
		ErrRsponse(c,errCode)
		return
	}
	if v, ok := param["userName"]; ok {
		userName = v
	} else {
		ErrRsponse(c,errCode)
		return
	}
	if v, ok := param["format"]; ok {
		format = v
	} else {
		ErrRsponse(c,errCode)
		return
	}
	if !file.IsOk(format) {
		ErrRsponse(c,errCode)
		return
	}
    //...
	//删除结尾
}
```

## 机遇

前几天在看[GIN-VUE-ADMIN](https://github.com/flipped-aurora/gin-vue-admin)代码的时候，偶然看到一个通过反射去做参数校验的方式。
嘿，学到了！

## 改变

### 定义规则

校验规则使用一个`map`存储，`key`为字段名，`value`为规则列表，并使用一个`string`类型的切片来存储。

> 后续计划加入`tag`标签定义规则的功能以及增加通过函数参数的方式，实现自定义规则校验

```go
type Rules map[string][]string
```

支持的规则有：
- 不为空
- 等于、不等于
- 大于、小于
- 大于等于、小于等于

> 对于数值类型为比较值大小，对于字符串或者切片等类型为比较长度大小

比如调用生成小于规则的方法，则会返回一个小于指定值规则的字符串，用于后面校验器使用

```go
// Lt <
func (verifier verifier) Lt(limit string) string {
	return fmt.Sprintf("%s%s%s", lt, verifier.separator, limit)
}
```

规则定义示例：

```go
	UserRequestRules = go_opv.Rules{
		"Name": {myVerifier.NotEmpty(), myVerifier.Lt("10")},
		"Age":  {myVerifier.Lt("100")},
	}
	//map[Age:[lt#100] Name:[notEmpty lt#10]]
```

规则含义为`Age`字段长度或值小于100，`Name`字段不为空且长度或值小于10。

### 验证器

先通过反射获取待检验参数的值和类型，判断是否为`struct`（目前只实现了对`struct`校验的功能，计划后续加入对`map`的校验功能），
获取`struct`属性数量并遍历所有属性，并遍历每个字段下所有规则，对定义的每一个规则进行校验是否合格。

```go
func (verifier verifier) Verify(st interface{}, rules Rules) (err error) {
	typ := reflect.TypeOf(st)
	val := reflect.ValueOf(st)

	if val.Kind() != reflect.Struct {
		return errors.New("expect struct")
	}
	num := val.NumField()
	//遍历需要验证对象的所有字段
	for i := 0; i < num; i++ {
		tagVal := typ.Field(i)
		val := val.Field(i)
		if len(rules[tagVal.Name]) > 0 {
			for _, v := range rules[tagVal.Name] {
				switch {
				case v == "notEmpty":
					if isEmpty(val) {
						return errors.New(tagVal.Name + " value can not be nil")
					}
				case verifier.conditions[strings.Split(v, verifier.separator)[0]]:
					if !compareVerify(val, v, verifier.separator) {
						return errors.New(tagVal.Name + " length or value is illegal," + v)
					}
				}
			}
		}
	}
	return nil
}
```

规则校验有两种，分别是判空 和条件校验。  
判空是通过反射`reflect.Value`获得字段值，并通过反射`value.Kind()`获得字段类型。
最终使用`switch`分别对不同类型 字段进行判断。

```go
func isEmpty(value reflect.Value) bool {
	switch value.Kind() {
	case reflect.String:
		return value.Len() == 0
	case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
		return value.Int() == 0
	//此处省略其他类型判断
	//...
	}
	return reflect.DeepEqual(value.Interface(), reflect.Zero(value.Type()).Interface())
}
```

条件校验则是通过开始时定义的范围条件进行校验，传入反射`reflect.Value`获得字段值，定义的规则，以及规则中的分隔符。先通过`switch`判断其类型，
再通过`switch`判断条件是大于小于或是其他条件，然后进行相应判断。

```go
func compareVerify(value reflect.Value, verifyStr, separator string) bool {
	switch value.Kind() {
	case reflect.String, reflect.Slice, reflect.Array:
		return compare(value.Len(), verifyStr, separator)
	//此处省略其他类型判断
	//...
	default:
		return false
	}
}
```

### 封装

为了调用方便，做了一层封装，使用[函数选项模式](https://ormissia.github.io/posts/knowledge/2021-07-22/)对校验器进行封装，使调用更为方便。

```go
var defaultVerifierOptions = verifierOptions{
	separator: ":",
	conditions: map[string]bool{
		eq: true,
		ne: true,
		gt: true,
		lt: true,
		ge: true,
		le: true,
	},
}

type VerifierOption func(o *verifierOptions)
type verifierOptions struct {
	conditions map[string]bool
	separator  string
}

// SetSeparator Default separator is ":".
func SetSeparator(seq string) VerifierOption {
	return func(o *verifierOptions) {
		o.separator = seq
	}
}

func SwitchEq(sw bool) VerifierOption {
	return func(o *verifierOptions) {
		o.conditions[eq] = sw
	}
}

//...
//此处省略其他参数的设置

type Verifier interface {
	Verify(obj interface{}, rules Rules) (err error)

	NotEmpty() string
	Ne(limit string) string
	Gt(limit string) string
	Lt(limit string) string
	Ge(limit string) string
	Le(limit string) string
}

type verifier struct {
	separator  string
	conditions map[string]bool
}

func NewVerifier(opts ...VerifierOption) Verifier {
	options := defaultVerifierOptions
	for _, opt := range opts {
		opt(&options)
	}
	return verifier{
		separator:  options.separator,
		conditions: options.conditions,
	}
}

//...
//此处省略接口的实现
```

### 发布

> 好了，基本功能完成了，如果仅仅是放在每个项目的`utils`拷来拷去，显然十分的不优雅。  
> 那么这就需要发布到[pkg.go.dev](https://pkg.go.dev)才能通过`go get`命令正常被其他项目所引用。

1. 首先是`git commit`、`git push`一把梭将项目整到`GitHub`上。
2. 由于[pkg.go.dev](https://pkg.go.dev)的版本管理机制需要给项目打上`tag`，`git tag v0.0.1`基础版本，😋先定个`0.0.1`吧，
然后`git push`再走一遍。
3. 当然这时候还没完，需要自己`go get`一下，加上`GitHub`仓库名执行一下`go get github.com/ormissia/go-opv`
4. 这样仓库就可以正常被引用了。而且用不了多久，就可以从[pkg.go.dev](https://pkg.go.dev)上搜到相应的项目了。
5. 最后贴一下次项目的连接：[go-opv](https://pkg.go.dev/github.com/ormissia/go-opv)

> 当然，这个过程中也遇到过小坑。项目中`go.mod`中的模块名需要写`GitHub`的仓库地址，对应此项目即为`module github.com/ormissia/go-opv`。
> 如果项目版本有更新，打了新的`tag`之后。可以通过`go get github.com/ormissia/go-opv@v0.0.3`拉取指定版本，目前尚不清楚
> [pkg.go.dev](https://pkg.go.dev)是否会自动同步`GitHub`上最新的`tag`。

## 检验

测试用例？  
好吧，`// TODO`

> 老铁看到底了，来个star吧😁  
> ↓↓↓↓↓↓↓↓↓  
> [GitHub仓库](https://github.com/ormissia/go-opv)
