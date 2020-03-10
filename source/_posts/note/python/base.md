---
title: python 基础
categories:
  - note
---

python 基础知识

<!--more-->

### 一、环境安装

- brew install python3
- 设置别名（~/.bash_profile）

```
alias python="/usr/local/Cellar/python/3.7.0/bin/python3.7"
alias pip="/usr/local/Cellar/python/3.7.0/bin/pip3.7"
```

- 安装 vscode python 插件
- 指定 vscode 环境变量 python.pythonPath

```
<!-- 输入 interpreter，查看解释器路径 -->
commond + shift + p
<!-- 设置环境变量值 -->
/usr/local/bin/python3
```

- 安装设置 virtualenv

```
<!-- 安装 -->
pip3 install virtualenv
<!-- 设置 -->
cd ~
virtualenv .pyenv
<!-- 设置环境变量 python.venvPath -->
~/.pyenv
```

### 二、基础知识

#### 2.1、注释

```
<!-- 单行注释 -->
#

<!-- 多行注释 -->
'''
'''
<!-- 或 -->
"""
"""
```

#### 2.2、保留字

```
<!-- Python 的标准库提供了一个 keyword 模块，可以输出当前版本的所有关键字 -->
>>> import keyword
>>> keyword.kwlist
<!-- 保留字列表 -->
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class',
 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from',
 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass',
 'raise', 'return', 'try', 'while', 'with', 'yield']
```

#### 2.3、行与缩进

缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数

```
if True:
    print ("Answer")
    print ("True")
else:
    print ("Answer")
  print ("False")    # 缩进不一致，会导致运行错误
```

#### 2.4、语句、输出、输入、del、换行

- 如果语句很长，我们可以使用反斜杠(\\)来实现多行语句
- print 默认输出是换行的，如果要实现不换行需要在变量末尾加上 end=''
- input 用户等待用户输入
- del 用于删除单个或多个对象的引用
- 可以在同一行中使用多条语句，语句之间使用分号(;)分割
- 制表符，增加字符缩进 \t
- 换行符，换行输出 \n

```
<!-- 多条语句同行显示 -->
import sys; x = 'runoob'; sys.stdout.write(x + '\n')
<!-- input 输入 -->
input("\n\n按下 enter 键后退出。")
```

#### 2.5、基本运算符

- 算术运算符
  - +
  - -
  - \*
  - / 浮点数除法，比如 10 / 2 = 5.0
  - // 除法取整，向下取接近除数的整数，比如 9 // 2 = 4，-9 // 2 = -5，7 // 2.0 = 3.0
  - \*\* N 次方，也可以用 pow() 方法
  - % 取模

```
<!-- 计算公式：r = a - n * (a // n) -->
print(-123 % 10) # 7
print(123 % -10) # -7
print(-123 % -10) # -3
```

- 比较运算符

  - 比较运算符返回 1 表示真，返回 0 表示假，与特殊的变量 True 和 False 等价，注意变量名的大写
  - ==
  - !=
  - \>
  - <
  - \>=
  - <=

- 赋值运算符

  - =
  - +=
  - -=
  - \*=
  - /=
  - %=
  - \*\*=
  - //=

- 位运算符

  - & 与
  - | 或
  - ^ 异或
  - ~ 取反
  - << 左移动
  - \>> 右移动

- 逻辑运算符

  - 假设变量 a 为 10, b 为 20:
  - and (a and b) 返回 20
  - or (a or b) 返回 10
  - not not(a and b) 返回 False

- 成员运算符

  - 比如：'a' in 'abc'，1 in [1, 2, 3]
  - in
  - not in

- 身份运算符

  - id() 函数用于获取对象内存地址，比如：a = 20; b = 20; print(a is b);
  - is 判断两个标识符是不是引用自一个对象，类似 id(x) == id(y)
  - is not 判断两个标识符是不是引用自不同对象，类似 id(a) != id(b)

- 运算符优先级

  - 以下列出了从最高到最低优先级的所有运算符：
  - \*\* 指数 (最高优先级)
  - ~ + - 按位翻转, 一元加号和减号 (最后两个的方法名为 +@ 和 -@)
  - \* / % // 乘，除，求余数和取整除
  - \+ \- 加法减法
  - \>> << 右移，左移运算符
  - & 位 'AND'
  - ^ | 位运算符
  - <= < > >= 比较运算符
  - == != 等于运算符
  - = %= /= //= -= += \*= \*\*= 赋值运算符
  - is is not 身份运算符
  - in not in 成员运算符
  - not and or 逻辑运算符

#### 2.6、流程控制

- 条件：if elif else
- 循环：
  - while ... else ...
  - for ... in ... else ...
  - break：跳出 for 和 while 的循环体，任何对应的循环 else 块将不执行
  - continue：跳过当前循环块中的剩余语句，然后继续进行下一轮循环
- pass：空语句，不做任何事情，一般用做占位语句，保持程序结构的完整性
