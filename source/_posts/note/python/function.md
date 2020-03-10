---
title: python 函数介绍
categories:
  - note
---

python 函数介绍

<!--more-->

### 二、内置函数

- range(start, stop[, step])，用于表示数字序列，内容不可变

```
# [0, 1, 2, 3, 4]，等价于 range(0, 5)
range(5)

# [0, 5, 10, 15, 20, 25]
range(0, 30, 5)  # 步长为 5
```

### 一、函数定义

- 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号 ()
- 任何传入参数和自变量必须放在圆括号中间，圆括号之间可以用于定义参数
- 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明
- 函数内容以冒号起始，并且缩进
- return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的 return 相当于返回 None

```
def 函数名（参数列表）:
    函数体
```

### 二、可更改(mutable)与不可更改(immutable)对象

- 不可变类型：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃，不是改变 a 的值，相当于新生成了 a
- 可变类型：变量赋值 la=[1,2,3,4] 后再赋值 la[2]=5 则是将 list la 的第三个元素值更改，本身 la 没有动，只是其内部的一部分值被修改了

python 函数的参数传递：

- 不可变类型：类似 c++ 的值传递，如 整数、字符串、元组。如 fun（a），传递的只是 a 的值，没有影响 a 对象本身。比如在 fun（a）内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身
- 可变类型：类似 c++ 的引用传递，如 列表、字典、集合。如 fun（la），则是将 la 真正的传过去，修改后 fun 外部的 la 也会受影响

python 中一切都是对象，严格意义我们不能说值传递还是引用传递，我们应该说传不可变对象和传可变对象

### 三、函数参数

- 必需参数

```
#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print (str)
   return

# 调用 printme 函数，不加参数会报错
printme()
```

- 关键字参数
  使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值

```
#可写函数说明
def printinfo( name, age ):
   "打印任何传入的字符串"
   print ("名字: ", name)
   print ("年龄: ", age)
   return

#调用printinfo函数
printinfo( age=50, name="runoob" )
```

- 默认参数

```
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print ("名字: ", name, " 年龄: ", age)
   return

#调用printinfo函数
printinfo( age=50, name="runoob" ) # 名字:  runoob  年龄:  50
printinfo( name="runoob" ) # 名字:  runoob  年龄:  35
```

- 不定长参数
  可能需要一个函数能处理比当初声明时更多的参数，这些参数叫做不定长参数

```
# 定义
def functionname([formal_args,] *var_args_tuple ):
   "函数_文档字符串"
   function_suite
   return [expression]

# 实例1（加了星号 * 的参数会以元组(tuple)的形式导入，存放所有未命名的变量参数）
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print (arg1, vartuple)
# 调用printinfo 函数
printinfo( 70, 60, 50 ) # 70 (60, 50)

# 实例2（* 解包传入列表）
a = [1, 2, 3]
def func(a, b, c):
    print(a, b, c)
func(*a) # 1 2 3

# 实例3（加了两个星号 ** 的参数会以字典的形式导入）
def printinfo( arg1, **vardict ):
   "打印任何传入的参数"
   print (arg1, vardict)
# 调用printinfo 函数
printinfo(1, a=2,b=3) # 1 {'a': 2, 'b': 3}

# 实例4（* 解包传入字典）
a = {'a': 1, 'b': 2, 'c': 3}
def func(a, b, c):
    print(a, b, c)
func(*a)  # a b c

# 实例5（** 解包传入字段）
a = {'a': 1, 'b': 2, 'c': 3}
def func(a, b, c):
    print(a, b, c)
func(**a)  # 1 2 3

# 实例6（声明函数时，参数中星号 * 可以单独出现，星号 * 后的参数必须用关键字传入）
def f(a,b,*,c):
    return a+b+c
f(1,2,3)   # 报错
f(1,2,c=3) # 正常

# * 和 ** 在表达式中应用
*range(5), 6, 7, 8 # (0, 1, 2, 3, 4, 6, 7, 8)
[*range(5), 6, 7, 8] # [0, 1, 2, 3, 4, 6, 7, 8]
{*range(3), 4, 5} # {0, 1, 2, 4, 5}
a = {'a': 1, 'b': 2, 'c': 3}
b = {'d': 4, 'e': 5, 'f': 6}
{**a, **b} # {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5, 'f': 6}
```

### 四、匿名函数

python 使用 lambda 来创建匿名函数。所谓匿名，意即不再使用 def 语句这样标准的形式定义一个函数：

- lambda 只是一个表达式，函数体比 def 简单很多
- lambda 的主体是一个表达式，而不是一个代码块。仅仅能在 lambda 表达式中封装有限的逻辑进去
- lambda 函数拥有自己的命名空间，且不能访问自己参数列表之外或全局命名空间里的参数
- 虽然 lambda 函数看起来只能写一行，却不等同于 C 或 C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率
- 语法：lambda [arg1 [,arg2,.....argn]]:expression

```
sum = lambda arg1, arg2: arg1 + arg2
# 调用sum函数
print ("相加后的值为 : ", sum( 10, 20 )) # 30
print ("相加后的值为 : ", sum( 20, 20 )) # 40
```

### 五、return 语句

用于退出函数，选择性地向调用方返回一个表达式。不带参数值的 return 语句返回 None。

```
def sum( arg1, arg2 ):
   total = arg1 + arg2
   print ("函数内 : ", total) # 30
   return total
total = sum( 10, 20 )
print ("函数外 : ", total) # 30
```

### 六、filter 函数

返回一个 filter 对象，其中包括对其执行函数时为真的所有元素

```
# 语法
filter(func, seq)

# 实例
l = range(0, 10)
res = filter(lambda item: item % 2 != 0, l)
print(list(res)) # [1, 3, 5, 7, 9]
```

### 七、map 函数

创建一个列表，其中包含对指定序列包含的项执行指定函数返回的值

```
# 语法
map(func, seq)

# 实例
l = range(0, 10)
res = map(lambda item: item * 2, l)
print(list(res)) # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 八、reduce 函数

使用指定的函数将序列的前两个元素合二为一，再将结果与第三个元素合二为一，依次类推，直到处理完整个序列并得到一个结果

```
# 语法
reduce(func, seq[, initial])

# 实例
from functools import reduce # python3 中 reduce 函数被取消了，放入到了 functools 模块中
l = range(0, 10)
res = reduce(lambda prev, item: prev + item, l)
print(res)  # 45
```
