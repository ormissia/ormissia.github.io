# Go 语法

#go #golang #语法 #编程语言

## 设置代理

```shell
go env -w GOPROXY=https://goproxy.cn,direct
```

## init () 函数

`init()` 函数的执行顺序：
- 对同一个 `go` 文件的 `init()` 调用顺序是从上到下的。
- 对同一个 `package` 中不同文件是按文件名字符串比较"从小到大"顺序调用各文件中的 `init()` 函数。
- 对于不同的 `package`，如果不相互依赖的话，按照 `main` 包中”先 `import` 的后调用”的顺序调用其包中的 `init()`，如果 `package` 存在依赖，则先调用最早被依赖的 `package` 中的 `init()`，最后调用 `main` 函数。
- _如果 `init` 函数中使用了 `println()` 或者 `print()` 你会发现在执行过程中这两个不会按照你想象中的顺序执行。这两个函数官方只推荐在测试环境中使用，对于正式环境不要使用。_

## 基本类型

| 类型         | 长度 | 默认值 | 说明                                      |
| ------------ | ---- | ------ | ----------------------------------------- |
| bool         | 1    | false  |                                           |
| byte         | 1    | 0      | uint8                                     |
| rune         | 4    | 0      | Unicode Code Point, int32                 |
| int/uint     | 4/8  | 0      | 32 或 64 位                               |
| int8/uint8   | 1    | 0      | -128 ~ 127, 0 ~ 255，byte是uint8 的别名   |
| int16/uint16 | 2    | 0      | -32768 ~ 32767, 0 ~ 65535                 |
| int32/uint32 | 4    | 0      | -21亿~ 21亿, 0 ~ 42亿，rune是int32 的别名 |
| int64/uint64 | 8    | 0      |                                           |
| float32      | 4    | 0.0    |                                           |
| float64      | 8    | 0.0    |                                           |
| complex64    | 8    |        |                                           |
| complex128   | 16   |        |                                           |
| uintptr      | 4/8  |        |                                           |
| array        |      |        |                                           |
| struct       |      |        |                                           |
| string       |      | ""     |                                           |
| slice        |      | nil    |                                           |
| map          |      | nil    |                                           |
| channel      |      | nil    |                                           |
| interface    |      | nil    | 接口                                      |
| function     |      | nil    | 函数                                      |

### 字符串转义符

| 转义符 | 含义                       |
| ------ | -------------------------- |
| \\r    | (回车符) 返回当前行行首    |
| \\n    | （换行符）向下移动一个字符 |
| \\t    | 制表符                     |
| '      | 单引号                     |
| "      | 双引号                     |
| /      | 斜杠                       |

## #切片

### 切片定义

- 使用 `make` 声明
```go
// make中后两个参数为len和cap，后者可以省略
s := make([]int, 0, 100)
```

```go
// len允许运行时期动态指定
func f() {  
   _ = make([]int, l())  
}  
  
func l() int {  
   return 1  
}
```
- 使用字面值声明
```go
// 中括号中不应有数字，加上数字则声明的是数组
s := []int{0, 1, 2, 3, 4, 5}
```
- 从现有的数组中声明
```go
s := [6]int{0, 1, 2, 3, 4, 5} 
ns1 := s[1:3:4]  
ns2 := s[1:3]  
ns3 := s[:3]  
ns4 := s[1:]  
ns5 := s[:]
```

### 切片的遍历

```go
s := []int{0, 1, 2, 3, 4}  
for i, a := range s {  
   fmt.Printf("addr_i: %p\taddr_a: %p\n", &s[i], &a)  
   // 此处注意：如果需要修改slice中的值，修改a的值是不起作用的，需要使用索引修改原切片值  
   a++  
   s[i] += 10  
}  
fmt.Println(s)
```

控制台打印：
```shell
addr_i: 0x140000a8030   addr_a: 0x140000a4008
addr_i: 0x140000a8038   addr_a: 0x140000a4008
addr_i: 0x140000a8040   addr_a: 0x140000a4008
addr_i: 0x140000a8048   addr_a: 0x140000a4008
addr_i: 0x140000a8050   addr_a: 0x140000a4008
[10 11 12 13 14]
```
> 根据打印可以看出：上述代码中变量 `a` 的地址始终是相同的

### 切片的传递

> `go` 中函数参数传递方式都为值传递。而对于切片来说，切片的结构中传递的是一个具体数组的地址。**如果函数中的切片发生扩容，那么扩容之后的操作对于函数外部是不可见的**。

```go

```

> 如果需要将改动传递到函数外部，要么将修改后的切片作为值返回，要么参数定义时使用切片指针。

```go
```

### 源码

> 源码参考 [go version go1.19.2 darwin/arm64](https://github.com/golang/go/tree/release-branch.go1.19)

-  [src/runtime/slice.go](https://github.com/golang/go/blob/release-branch.go1.19/src/runtime/slice.go)

#### 切片数据结构

```go
type slice struct {  
   array unsafe.Pointer  
   len   int  
   cap   int  
}
```

#### 创建切片

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {  
   mem, overflow := math.MulUintptr(et.size, uintptr(cap))  
   if overflow || mem > maxAlloc || len < 0 || len > cap {  
      // NOTE: Produce a 'len out of range' error instead of a  
      // 'cap out of range' error when someone does make([]T, bignumber).      // 'cap out of range' is true too, but since the cap is only being      // supplied implicitly, saying len is clearer.      // See golang.org/issue/4085.      mem, overflow := math.MulUintptr(et.size, uintptr(len))  
      if overflow || mem > maxAlloc || len < 0 {  
         panicmakeslicelen()  
      }  
      panicmakeslicecap()  
   }  
  
   return mallocgc(mem, et, true)  
}  

// 64位
func makeslice64(et *_type, len64, cap64 int64) unsafe.Pointer {  
   len := int(len64)  
   if int64(len) != len64 {  
      panicmakeslicelen()  
   }  
  
   cap := int(cap64)  
   if int64(cap) != cap64 {  
      panicmakeslicecap()  
   }  
  
   return makeslice(et, len, cap)  
}
```

#### 切片扩容

```go
// growslice handles slice growth during append.// It is passed the slice element type, the old slice, and the desired new minimum capacity,  
// and it returns a new slice with at least that capacity, with the old data// copied into it.  
// The new slice's length is set to the old slice's length,  
// NOT to the new requested capacity.  
// This is for codegen convenience. The old slice's length is used immediately  
// to calculate where to write new values during an append.// TODO: When the old backend is gone, reconsider this decision.  
// The SSA backend might prefer the new length or to return only ptr/cap and save stack space.func growslice(et *_type, old slice, cap int) slice {  
   if raceenabled {  
      callerpc := getcallerpc()  
      racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, abi.FuncPCABIInternal(growslice))  
   }  
   if msanenabled {  
      msanread(old.array, uintptr(old.len*int(et.size)))  
   }  
   if asanenabled {  
      asanread(old.array, uintptr(old.len*int(et.size)))  
   }  
  
   if cap < old.cap {  
      panic(errorString("growslice: cap out of range"))  
   }  
  
   if et.size == 0 {  
      // append should not create a slice with nil pointer but non-zero len.  
      // We assume that append doesn't need to preserve old.array in this case.      return slice{unsafe.Pointer(&zerobase), old.len, cap}  
   }  
  
   newcap := old.cap  
   doublecap := newcap + newcap  
   if cap > doublecap {  
      newcap = cap  
   } else {  
      const threshold = 256  
      if old.cap < threshold {  
         newcap = doublecap  
      } else {  
         // Check 0 < newcap to detect overflow  
         // and prevent an infinite loop.         for 0 < newcap && newcap < cap {  
            // Transition from growing 2x for small slices  
            // to growing 1.25x for large slices. This formula            // gives a smooth-ish transition between the two.            newcap += (newcap + 3*threshold) / 4  
         }  
         // Set newcap to the requested cap when  
         // the newcap calculation overflowed.         if newcap <= 0 {  
            newcap = cap  
         }  
      }  
   }  
  
   var overflow bool  
   var lenmem, newlenmem, capmem uintptr  
   // Specialize for common values of et.size.  
   // For 1 we don't need any division/multiplication.   // For goarch.PtrSize, compiler will optimize division/multiplication into a shift by a constant.   // For powers of 2, use a variable shift.   switch {  
   case et.size == 1:  
      lenmem = uintptr(old.len)  
      newlenmem = uintptr(cap)  
      capmem = roundupsize(uintptr(newcap))  
      overflow = uintptr(newcap) > maxAlloc  
      newcap = int(capmem)  
   case et.size == goarch.PtrSize:  
      lenmem = uintptr(old.len) * goarch.PtrSize  
      newlenmem = uintptr(cap) * goarch.PtrSize  
      capmem = roundupsize(uintptr(newcap) * goarch.PtrSize)  
      overflow = uintptr(newcap) > maxAlloc/goarch.PtrSize  
      newcap = int(capmem / goarch.PtrSize)  
   case isPowerOfTwo(et.size):  
      var shift uintptr  
      if goarch.PtrSize == 8 {  
         // Mask shift for better code generation.  
         shift = uintptr(sys.Ctz64(uint64(et.size))) & 63  
      } else {  
         shift = uintptr(sys.Ctz32(uint32(et.size))) & 31  
      }  
      lenmem = uintptr(old.len) << shift  
      newlenmem = uintptr(cap) << shift  
      capmem = roundupsize(uintptr(newcap) << shift)  
      overflow = uintptr(newcap) > (maxAlloc >> shift)  
      newcap = int(capmem >> shift)  
   default:  
      lenmem = uintptr(old.len) * et.size  
      newlenmem = uintptr(cap) * et.size  
      capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))  
      capmem = roundupsize(capmem)  
      newcap = int(capmem / et.size)  
   }  
  
   // The check of overflow in addition to capmem > maxAlloc is needed  
   // to prevent an overflow which can be used to trigger a segfault   // on 32bit architectures with this example program:   //   // type T [1<<27 + 1]int64   //   // var d T   // var s []T   //   // func main() {   //   s = append(s, d, d, d, d)   //   print(len(s), "\n")   // }   if overflow || capmem > maxAlloc {  
      panic(errorString("growslice: cap out of range"))  
   }  
  
   var p unsafe.Pointer  
   if et.ptrdata == 0 {  
      p = mallocgc(capmem, nil, false)  
      // The append() that calls growslice is going to overwrite from old.len to cap (which will be the new length).  
      // Only clear the part that will not be overwritten.      memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)  
   } else {  
      // Note: can't use rawmem (which avoids zeroing of memory), because then GC can scan uninitialized memory.  
      p = mallocgc(capmem, et, true)  
      if lenmem > 0 && writeBarrier.enabled {  
         // Only shade the pointers in old.array since we know the destination slice p  
         // only contains nil pointers because it has been cleared during alloc.         bulkBarrierPreWriteSrcOnly(uintptr(p), uintptr(old.array), lenmem-et.size+et.ptrdata)  
      }  
   }  
   memmove(p, old.array, lenmem)  
  
   return slice{p, old.len, newcap}  
}
```

## `map`

### map 定义

```go
m1 := make(map[string]int, 100)   
  
m2 := map[string]int{  
   "a": 1,  
   "b": 2,  
}
```

### map 的基本使用

```go
aValue := m["a"]  

// 第二个变量ok可用于判断当前key是否存在
bValue, ok := m["b"]
```

## map 的遍历

```go
for k, v := range m {  
   fmt.Println(k,v)  
}
```

### 删除元素

```go
delete(m, "a")
```

### 源码

> 源码参考 [go version go1.19.2 darwin/arm64](https://github.com/golang/go/tree/release-branch.go1.19)

-  [src/runtime/map.go](https://github.com/golang/go/blob/release-branch.go1.19/src/runtime/map.go)

#### #hash冲突

- #开放定址法 ：当要存储一对 `kv` ，发现 `hash(key)` 的下表已经被别的 `key` 占用时，就在这个数组中重新找空白没有被占用的位置存储这个 `key`。常见的有：`线性探测法`，`线性补偿探测法`， `随机探测法`。
- #拉链法 ：可以简单理解成数组中元素指向链表的头结点的一种结构。当 `key` 发生 `hash` 冲突时，在冲突位置的元素上形成一个链表。当查找时，发现 `key` 冲突，顺着链表一直往下找，直到链表的尾节点，找不到则返回空。

开放定址法和拉链法的优缺点：
- 拉链法通常比线性探测法处理简单
- 线性探测查找是会被拉链法会更消耗时间
- 线性探测会更加容易导致扩容，而拉链不会
- 拉链存储了指针，所以空间上会比线性探测占用多一点
- 拉链是动态申请存储空间的，所以更适合链长不确定的

#### map 的结构

```go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}

// mapextra holds fields that are not present on all maps.
type mapextra struct {
	// If both key and elem do not contain pointers and are inline, then we mark bucket
	// type as containing no pointers. This avoids scanning such maps.
	// However, bmap.overflow is a pointer. In order to keep overflow buckets
	// alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
	// overflow and oldoverflow are only used if key and elem do not contain pointers.
	// overflow contains overflow buckets for hmap.buckets.
	// oldoverflow contains overflow buckets for hmap.oldbuckets.
	// The indirection allows to store a pointer to the slice in hiter.
	overflow    *[]*bmap
	oldoverflow *[]*bmap

	// nextOverflow holds a pointer to a free overflow bucket.
	nextOverflow *bmap
}

// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```

#### map set 以及扩容

```go
// Like mapaccess, but allocates a slot for the key if it is not present in the map.  
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {  
   if h == nil {  
      panic(plainError("assignment to entry in nil map"))  
   }  
   if raceenabled {  
      callerpc := getcallerpc()  
      pc := abi.FuncPCABIInternal(mapassign)  
      racewritepc(unsafe.Pointer(h), callerpc, pc)  
      raceReadObjectPC(t.key, key, callerpc, pc)  
   }  
   if msanenabled {  
      msanread(key, t.key.size)  
   }  
   if asanenabled {  
      asanread(key, t.key.size)  
   }  
   if h.flags&hashWriting != 0 {  
      fatal("concurrent map writes")  
   }  
   hash := t.hasher(key, uintptr(h.hash0))  
  
   // Set hashWriting after calling t.hasher, since t.hasher may panic,  
   // in which case we have not actually done a write.   h.flags ^= hashWriting  
  
   if h.buckets == nil {  
      h.buckets = newobject(t.bucket) // newarray(t.bucket, 1)  
   }  
  
again:  
   bucket := hash & bucketMask(h.B)  
   if h.growing() {  
      growWork(t, h, bucket)  
   }  
   b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))  
   top := tophash(hash)  
  
   var inserti *uint8  
   var insertk unsafe.Pointer  
   var elem unsafe.Pointer  
bucketloop:  
   for {  
      for i := uintptr(0); i < bucketCnt; i++ {  
         if b.tophash[i] != top {  
            if isEmpty(b.tophash[i]) && inserti == nil {  
               inserti = &b.tophash[i]  
               insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))  
               elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))  
            }  
            if b.tophash[i] == emptyRest {  
               break bucketloop  
            }  
            continue  
         }  
         k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))  
         if t.indirectkey() {  
            k = *((*unsafe.Pointer)(k))  
         }  
         if !t.key.equal(key, k) {  
            continue  
         }  
         // already have a mapping for key. Update it.  
         if t.needkeyupdate() {  
            typedmemmove(t.key, k, key)  
         }  
         elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))  
         goto done  
      }  
      ovf := b.overflow(t)  
      if ovf == nil {  
         break  
      }  
      b = ovf  
   }  
  
   // Did not find mapping for key. Allocate new cell & add entry.  
  
   // If we hit the max load factor or we have too many overflow buckets,   // and we're not already in the middle of growing, start growing.   if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {  
      hashGrow(t, h)  
      goto again // Growing the table invalidates everything, so try again  
   }  
  
   if inserti == nil {  
      // The current bucket and all the overflow buckets connected to it are full, allocate a new one.  
      newb := h.newoverflow(t, b)  
      inserti = &newb.tophash[0]  
      insertk = add(unsafe.Pointer(newb), dataOffset)  
      elem = add(insertk, bucketCnt*uintptr(t.keysize))  
   }  
  
   // store new key/elem at insert position  
   if t.indirectkey() {  
      kmem := newobject(t.key)  
      *(*unsafe.Pointer)(insertk) = kmem  
      insertk = kmem  
   }  
   if t.indirectelem() {  
      vmem := newobject(t.elem)  
      *(*unsafe.Pointer)(elem) = vmem  
   }  
   typedmemmove(t.key, insertk, key)  
   *inserti = top  
   h.count++  
  
done:  
   if h.flags&hashWriting == 0 {  
      fatal("concurrent map writes")  
   }  
   h.flags &^= hashWriting  
   if t.indirectelem() {  
      elem = *((*unsafe.Pointer)(elem))  
   }  
   return elem  
}
```

## `range`



## #泛型

### 泛型示例

```go
// Sum 使用泛型
func Sum[T int64 | float64](a, b T) T {  
    return a+b  
}
```

### 自定义泛型类型

```go
// CustomInt 泛型与接口声明类似  type CustomInt interface {  
   int | int8 | int16 | int32 | int64  
}  
  
// Sum T的类型为声明的CustomInt  func Sum[T CustomInt](a, b T) T {  
   return a+b  
}
```

> 成员类型支持 `go` 中所有基本类型

```go
type CustomT interface {  
   int | float32 | bool | chan int | map[int]int | [10]int | []int | struct{} | *http.Client  
}
```

> 泛型中的 `~` 符号都是与类型一起出现的，用来表示支持该类型的衍生类型

```go
// int8的衍生类型  
type int8A int8  
type int8B = int8  
  
// CustomInt 不仅支持int8, 还支持int8的衍生类型int8A和int8B  
type CustomInt interface {  
   ~int8  
}
```

### 使用带泛型的函数

```go
var a int = 10  
var b int = 20  
  
// 方法1，正常调用，编译器会自动推断出传入类型是int
Sum(a, b)  
  
// 方法2，显式告诉函数传入的类型是int
Sum[int](a, b)
```

### 内置的泛型类型 `any` 和 `comparable`

> `any`: 表示  `go` 里面所有的内置基本类型，等价于 `interface{}`

```go
// any is an alias for interface{} and is equivalent to interface{} in all ways.  
type any = interface{}
```

> `comparable`: 表示 `go` 里面所有内置的可比较类型：`int`、`uint`、`float`、`bool`、`struct`、指针等一切可以比较的类型

```go
// comparable is an interface that is implemented by all comparable types  
// (booleans, numbers, strings, pointers, channels, arrays of comparable types,  
// structs whose fields are all comparable types).  
// The comparable interface may only be used as a type parameter constraint,  
// not as the type of a variable.  
type comparable interface{ comparable }
```

### 泛型与结构体

```go
type AgeT interface {  
   int8 | int16  
}  
  
type NameE interface {  
   string  
}  
  
type User[T AgeT, E NameE] struct {  
   age  T  
   name E  
}  
  
// GetAge 获取年龄  func (u *User[T, E]) GetAge() T {  
   return u.age  
}  
  
// GetName 获取名字  func (u *User[T, E]) GetName() E {  
   return u.name  
}
```

```go
// 声明要使用的泛型的类型  var u User[int8, string]  
  
// 赋值  u.age = 10  
u.name = "ormissia"  
  
// 调用方法  age := u.GetAge()  
name := u.GetName()  
  
fmt.Println(age, name)
// 10 ormissia
```

### 泛型与 `switch`

> 泛型和 `switch` 配合使用时无法通过编译，只能先将泛型赋值给 `interface` 才可以和 `switch` 配合使用

```go
// GetX 编译不通过  func GetX[T any]() T {  
   var t T  
  
   switch T {  
   case int:  
      t = 18  
   }  
  
   return t  
}  
  
// Get 编译通过  func Get[T any]() T {  
   var t T  
  
   var ti interface{} = &t  
   switch v := ti.(type) {  
   case *int:  
      *v = 18  
   }  
  
   return t  
}
```

### 泛型的实际使用
   - [缓存适配器中泛型的使用](https://github.com/ormissia/cache_shim/blob/b295ded156d37f6a20d8811e740736a8c228cdbf/cache_strategy.go#L24) [[计算机/项目/Go缓存垫片]]

## panic 与 recovery

- 调用 `panic` 后会立刻停止执行当前函数的剩余代码，并在当前 Goroutine 中递归执行调用方的 `defer`
-  `recover` 可以中止 `panic` 造成的程序崩溃。它是一个只能在 `defer` 中发挥作用的函数，在其他作用域中调用不会发挥作用

## channel

## switch/case

- 单个 `case` 语句中，可以出现多个结果选项
- 只有在 `case` 中明确添加 `fallthrough` 关键字，才会明确执行紧跟的下一个`case`