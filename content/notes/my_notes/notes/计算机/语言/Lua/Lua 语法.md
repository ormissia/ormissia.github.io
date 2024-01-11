# Lua 语法

#lua #编程语言 

## 注释

- 单行注释
```lua
-- 单行注释
```

- 多行注释
```lua
--[[  
多行注释  
多行注释  
]]
```

## 数据类型

> Lua 是动态类型语言，变量不要类型定义，只需要为变量赋值

Lua 中有 8 个基本类型分别为：`nil`、`boolean`、`number`、`string`、`userdata`、`function`、`thread` 和 `table`。

| 数据类型 | 描述                                                                                                                                                                                               |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nil      | 值nil属于该类，表示一个无效值（在条件表达式中相当于false）                                                                                                                                         |
| boolean  | 包含两个值：false 和 true                                                                                                                                                                          |
| number   | 表示双精度类型的实浮点数                                                                                                                                                                           |
| string   | 字符串由一对双引号或单引号来表示                                                                                                                                                                   |
| function | 由 C 或 Lua 编写的函数                                                                                                                                                                             |
| userdata | 表示任意存储在变量中的C数据结构                                                                                                                                                                    |
| thread   | 表示执行的独立线路，用于执行协同程序                                                                                                                                                               |
| table    | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。<br>在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。 |

可以使用 type 函数测试给定变量或者值的类型：
```lua
print(type("Hello world"))  
print(type(10.4 * 3))  
print(type(print))  
print(type(type))  
print(type(true))  
print(type(nil))  
print(type(type(X)))

--[[
输出
string  
number  
function  
function  
boolean  
nil  
string  
]]
```

## 变量

> 在默认情况下，变量总是认为是全局的，哪怕是语句块或是函数里，除非用 local 显式声明为局部变量

Lua 变量有三种类型：全局变量、局部变量、表中的域
变量的默认值均为 nil

```lua
a = 5               -- 全局变量  
local b = 5         -- 局部变量
```

## 循环

### while

```lua
a=10  
while( a < 20 )  
do  
   print("a 的值为:", a)  
   a = a+1  
end
```

### for

#### 数值 for 循环

```lua
for i=10,1,-1 do  
    print(i)  
end
```

> for 的三个表达式在循环开始前一次性求值，以后不再进行求值。比如上面的 f (x) 只会在循环开始前执行一次，其结果用在后面的循环中

```lua
function f(x)  
    print("function")  
    return x*2  
end  
for i=1,f(5) do print(i)  
end
```

#### 泛型 for 循环

> i 是数组索引值，v 是对应索引的数组元素值。ipairs 是 Lua 提供的一个迭代器函数，用来迭代数组，相当于 Go 里面的 range

```lua
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end
```

### repeat... until

> 相当于某些语言里的 `do...while`

```lua
--[ 变量定义 --]  
a = 10  
--[ 执行循环 --]  
repeat  
   print("a的值为:", a)  
   a = a + 1  
until( a > 15 )
```

### 嵌套循环

```lua
j =2  
for i=2,10 do  
   for j=2,(i/j) , 2 do  
      if(not(i%j))  
      then  
         break  
      end  
      if(j > (i/j))then  
         print("i 的值为：",i)  
      end  
   end  
end
```

### 循环控制语句

- break：退出当前循环或语句，并开始脚本执行紧接着的语句

- goto
```lua
local a = 1  
::label:: print("--- goto label ---")  
  
a = a+1  
if a < 3 then  
    goto label   -- a 小于 3 的时候跳转到标签 label  
end
```

### 无限循环

```lua
while( true )  
do  
   print("循环将永远执行下去")  
end
```

## 流程控制

### if

```lua
--[ 定义变量 --]
a = 10;

--[ 使用 if 语句 --]
if( a < 20 )
then
   --[ if 条件为 true 时打印以下信息 --]
   print("a 小于 20" );
end
print("a 的值为:", a);
```

### if... else

```lua
--[ 定义变量 --]  
a = 100;  
--[ 检查条件 --]  
if( a < 20 )  
then  
   --[ if 条件为 true 时执行该语句块 --]  
   print("a 小于 20" )  
else  
   --[ if 条件为 false 时执行该语句块 --]  
   print("a 大于 20" )  
end  
print("a 的值为 :", a)
```

## 函数

函数定义格式：
```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```

- `optional_function_scope`：该参数是可选的指定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 `local`
- `function_name`：指定函数名称
- `argument1, argument2, argument3..., argumentn`：函数参数，多个参数以逗号隔开，函数也可以不带参数
- `function_body`：函数体，函数中需要执行的代码语句块
- `result_params_comma_separated`：函数返回值，Lua 语言函数可以返回多个值，每个值以逗号隔开

```lua
--[[ 函数返回两个值的最大值 --]]  
function max(num1, num2)  
  
   if (num1 > num2) then  
      result = num1;  
   else  
      result = num2;  
   end  
  
   return result;  
end  
-- 调用函数  
print("两值比较最大值为 ",max(10,4))  
print("两值比较最大值为 ",max(5,6))
```

### 多返回值

```lua
function maximum (a)  
    local mi = 1             -- 最大值索引  
    local m = a[mi]          -- 最大值  
    for i,val in ipairs(a) do  
       if val > m then  
           mi = i  
           m = val  
       end  
    end  
    return m, mi  
end  
  
print(maximum({8,10,23,12,5}))
```

### 可变参数

Lua 函数可以接受可变数目的参数，在函数参数列表中使用三点 ... 表示函数有可变的参数。

```lua
function average(...)  
   result = 0  
   local arg={...}    --> arg 为一个表，局部变量  
   for i,v in ipairs(arg) do  
      result = result + v  
   end  
   print("总共传入 " .. #arg .. " 个数")  
   return result/#arg  
end  
  
print("平均值为",average(10,5,3,4,5,6))
```

## 运算符

### 算术运算符

| 操作符 | 描述                 |
| ------ | -------------------- |
| +      | 加                   |
| -      | 减                   |
| *      | 乘                   |
| /      | 除                   |
| %      | 取余                 |
| ^      | 乘幂                 |
| -      | 负号                 |
| //     | 整除运算符(>=lua5.3) |

### 关系运算符

| 操作符 | 描述       |
| ------ | ---------- |
| ==     | 等于       |
| **~=** | **不等于** |
| >      | 大于       |
| <      | 小于       |
| >=     | 大于等于   |
| <=     | 小于等于   |

### 逻辑运算符

| 操作符 | 描述   |
| ------ | ------ |
| and    | 逻辑与 |
| or     | 逻辑或 |
| not    | 逻辑非 |

> 这里可以实现三目运算符的操作：`print(a > 10 and "yes" or "no")`

### 其他运算符

| 操作符 | 描述                 |
| ------ | -------------------- |
| ..     | 连接两个字符串       |
| #      | 返回字符串或表的长度（#"Hello" 返回 5） | 

### 运算符优先级

由高到低的顺序：
```
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or
```

## 字符串

Lua 语言中字符串可以使用以下三种方式来表示：

- 单引号间的一串字符
- 双引号间的一串字符
- `[[` 与 `]]` 间的一串字符

## 数组

**在 lua 中数组下标是从 1 开始的，有别于其他语言中从 0 开始**

### 一维数组

```lua
array = {"Lua", "Tutorial"}  
  
for i= 0, 2 do  
   print(array[i])  
end
```

### 多维数组

```lua
-- 初始化数组  
array = {}  
for i=1,3 do  
   array[i] = {}  
      for j=1,3 do  
         array[i][j] = i*j  
      end  
end  
  
-- 访问数组  
for i=1,3 do  
   for j=1,3 do  
      print(array[i][j])  
   end  
end
```

