---
layout: posts
title: Lua学习笔记二
date: 2019-04-15 21:00:34
Categories: 学习笔记
tags: [学习笔记, Lua]
---

# 循环

## while

```lua
while(条件)
do
  循环体
end
```

代码示例及结果：

```lua
a = 0
while(a < 10)
do
	print(a)
	a = a + 1
end
```

```lua
0
1
2
3
4
5
6
7
8
9
```

## for

```lua
for var=值1,值2,值2 do
  循环体
end
```

从值1变化到值2，每次变化以值3为步长，执行一次循环体。表达式3不指定时默认为1.

代码示例及结果：

```lua
for i = 0,10 do
  print(i)
end
```

```
0
1
2
3
4
5
6
7
8
9
10
```

变化范围包含值2

for 遍历table:

```lua
table = {"a","b","c"}
for i,v in ipairs(table) do
  print(i,v)
end
```

```lua
1	a
2	b
3	c
```

## repeat…until

```lua
repeat
  循环体
until(条件语句)
```

执行循环体，直到条件语句成立

代码示例及结果：

```lua
i = 0
repeat
  print(i)
  i = i+1
until(i>10)
```

```lua
0
1
2
3
4
5
6
7
8
9
10
```

和其他语言一样，break 可以跳出循环。

# 判断语句

## if

```lua
if(条件语句)
then
  	执行语句
end
```

代码示例及结果：

```lua
a = 10
if(a>0)
then
  print("a>0")
end
```

`a>0`

## if…else

```lua
if(条件语句)
then
  条件为真时语句
else
  条件为假时语句
end
```

代码示例及结果：

```lua
a = 10
if(a> 20)
then
  print("a>20")
else
  print("a<=20")
end
```

`a<=20`

## if…elseif…else

```lua
if(条件一)
then
  条件一为真执行
elseif(条件二)
then
  条件二为真执行
elseif(条件N)
then
  条件N为真执行
else
  以上都不满足的条件执行
end
```

代码示例及结果：

```lua
local score = 55
if(score >= 80 and score <= 100)
then
	print("A")
elseif(score < 80 and score>= 60)
then
	print("B")
else
	print("C")
end
```

`c`

# 函数

## 函数定义

```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```

+ optional_function_scope 指定函数是全局还是局部函数，默认不写是全局，局部是local
+ function_name 函数名
+ argument 参数列表
+ function_body 函数体
+ result_params_comma_separated 返回值，可以返回多个值，逗号隔开

```lua
--求两个数的最大值函数
function max(num1,num2)
  if(num1 > num2)
  then
    result = num1
  else
    result = num2
  end
  return result
end
```

lua 中可以将函数作为参数传递，作为回调函数：

```lua
--登录回调
callback = function(errorId)
  print("errorId:"..errorId)
end
--登录方法
function login(userName,passworld,loginCallback)
  if(userName == "123" and passworld == "123456") then
    loginCallback(0)
  else
    loginCallback(-1)
 	end
end
login("123","123456",callback) 
```

## 返回多值

如字符换查找，返回开始和结束的位置：(索引从1开始)

```lua
s,e = string.find("hello world","world")
print(s,e)
```

```lua
function maxinum(array)
  local index = 1
  local max = array[index]
  for i,v in ipairs(array) do
    if(v > max) then
      index = i
      max = v
    end
  end
  return index,max
end
index,max = maxinum({3,77,34,22,566,7,32,6})
print(index,max)
```

`5	566`

## 可变参数

Lua中可变参数和c,java中一样，用三个点...表示

```lua
function add(...)
	sum = 0
	for i,v in ipairs{...} do	--{...}表示可变参数的数组
		sum = sum + v
	end
	return sum
end
print(add(1,2,3,4))
```

```lua
local arg = {...} -- 将可变参数赋值给局部变量
length = #arg	--求可变参数的参数数量

select('#',...) --求可变参数的参数数量
select(n,...)   -- 访问 n 到 select('#',…) 的参数(多个值)
```



