# Go 缓存垫片

#go #缓存 #框架 #延时双删

> 一个数据库与内存之间的缓存适配器，运用了控制反转的思想。引入之后无需重复为每种实体实现相同的缓存策略

## 项目背景

大多数情况下我们在代码中添加 #Redis 作为缓存中间件之后都会多出很多代码来。每给一个实体添加缓存，都需要加一大堆代码。久而久之，代码库变得非常冗杂而且难看。

举个栗子，一个带缓存的实体的查询逻辑的伪代码：
```go
func (e *TestS) Select() {
	e, ok := SelectFromRedis()
	if !ok {
		et, ok := SelectFromDB()
		if !ok {
			return
		}
		e = et

		go func() {
			SaveToCache(e)
		}()

		return
	}
	return
}
```

假如一个服务中有多个需要维护到缓存中的实体，再加上删除，修改等其他逻辑，就会出现非常多的相似代码，导致维护十分麻烦，而且复用性极低。即使写成工具类，当有新服务加入时，也需要将代码拷来拷去，显得十分不专业。

因此，写了这个缓存适配器框架，使用 #控制反转 的思想，代码中的实体只需实现指定接口，删除、修改、查询时只需要调用该库的对应方法即可，无需关心具体实现，框架会自动完成缓存相关逻辑。

## 使用示例

1. 缓存客户端实现`CacheClintImpl`接口
```go
type CacheClintImpl interface {
	Del(key string) (int64, error)
	SetString(key, value string) error
	GetString(k string) (string, error)
}
```
实现参考[redis_storage.go](https://github.com/ormissia/cache_shim/blob/master/example/redis_ex/redis_storage.go)

2. 初始化缓存客户端并将缓存客户端示例初始化到框架中
```go
redis_ex.Init()
cache_shim.InitCacheClient(&redis_ex.RDB)
```
3. 需要做缓存的实体实现`CacheTypeImpl`接口
```go
// CacheType 需要缓存的实体接口定义  
type CacheType interface {  
   CacheKey() string  
   Expiration() int  
  
   Delete() error  
   Select() error  
   Update() error  
}
```

4. 需要做缓存的实体实现 `CacheTypeImpl` 接口
	因为控制反转，这个时候不需要直接调用实体的 `Select()` 方法，直接调用 `cache_shim` 包中的 `Select()` 函数即可，框架将自动维护缓存中的数据。
```go
_ = t1.Insert()
t, err := cache_shim.Select[*db_ex.UserEx](t1)
fmt.Printf("t.type: %T\tt: %v\terr: %v",t,t,err)
```
5. 向数据库中插入数据之后，使用框架提供的查询方法查询
	代码参考[main.go](https://github.com/ormissia/cache_shim/blob/master/example/main.go)

> 修改、删除同理

## 链接

- [项目源码](https://github.com/ormissia/cache_shim)