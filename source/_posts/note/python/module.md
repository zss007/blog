---
title: python 模块
categories:
  - note
---

python 模块

<!--more-->

### 一、定义

模块是一个包含所有你定义的函数和变量的文件，其后缀名是 .py。模块可以被别的程序引入，以使用该模块中的函数等功能。这也是使用 python 标准库的方法。

```
# 文件名: using_sys.py
import sys

print('命令行参数如下:')
for i in sys.argv:
   print(i)

print('\n\nPython 路径为：', sys.path, '\n')
```

- import sys 引入 python 标准库中的 sys.py 模块；这是引入某一模块的方法。
- sys.argv 是一个包含命令行参数的列表。
- sys.path 包含了一个 Python 解释器自动查找所需模块的路径的列表，其中第一项是当前目录。

### 二、import

想使用 Python 源文件，只需在另一个源文件里执行 import 语句。一个模块只会被导入一次，不管你执行了多少次 import。这样可以防止导入模块被一遍又一遍地执行。
编码时，import 顺序推荐为：标准库、第三方库包、自定义的包/模块。
在函数内部也可以 import 包，不过函数内部引用的包不能被函数外调用。

```
# 语法，将整个模块导入
import module1[, module2[,... moduleN]

# from 语句让你从模块中导入指定的部分到当前命名空间中
from modname import name1[, name2[, ... nameN]]

# 指定别名
from modname import name1 as rename

# 把一个模块的所有内容全都导入到当前的命名空间，不推荐，不直观，直接调用模块方法会给用户带来困惑
from modname import *
```

### 三、`__name__`属性

一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用`__name__`属性来使该程序块仅在该模块自身运行时执行。

```
# Filename: using_name.py
if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')

python using_name.py # 程序自身在运行
import using_name # 我来自另一模块
```

每个模块都有一个`__name__`属性，当其值是`__name__`时，表明该模块自身在运行，否则是被引入。

### 四、dir() 函数

内置的函数 dir() 可以找到模块内定义的所有名称，以一个字符串列表的形式返回:

```
import fibo
dir(fibo) # ['__name__', '__file__', 'fib', 'fib2']，其中 __file__ 表示文件全路径
fibo.__file__ # /Users/songsong.zhang/study/python-test/fibo.py

# 如果没有给定参数，那么 dir() 函数会罗列出当前定义的所有名称:
a = [1, 2, 3, 4, 5]
import fibo
fib = fibo.fib
dir() # 得到一个当前模块中定义的属性列表，['__builtins__', '__name__', 'a', 'fib', 'fibo', 'sys']
a = 5 # 建立一个新的变量 'a'
dir() # ['__builtins__', '__doc__', '__name__', 'a', 'sys']
del a # 删除变量名a
dir() # ['__builtins__', '__doc__', '__name__', 'sys']
```

### 五、package

- 定义
  包是一种管理 Python 模块命名空间的形式，采用"点模块名称"。比如一个模块的名称是 A.B，那么他表示一个包 A 中的子模块 B。采用点模块名称这种形式也不用担心不同库之间的模块重名的情况，这样不同的作者都可以提供 NumPy 模块，或者是 Python 图形库。
- 导入
  在导入一个包的时候，依次从当前包、内置标准库、sys.path（环境变量）查找包中包含的子目录。目录只有包含一个叫做 `__init__.py` 的文件才会被认作是一个包，主要是为了避免一些滥俗的名字（比如叫做 string）不小心的影响搜索路径中的有效模块。
  最简单的情况，放一个空的 `:file:__init__.py` 就可以了。当然这个文件中也可以包含一些初始化代码或者为 `__all__` 变量赋值。
  导入语句遵循如下规则：如果包定义文件 `__init__.py` 存在一个叫做 `__all__` 的列表变量，那么在使用 from package import \* 的时候就把这个列表中的所有名字作为包内容导入，如：

```
# sounds/effects/__init__.py
__all__ = ["echo", "surround", "reverse"] # 当你使用from sound.effects import * 这种用法时，你只会导入包里面这三个子模块
```

如果 `__all__` 真的没有定义，那么使用 from sound.effects import \* 这种语法的时候，就不会导入包 sound.effects 里的任何子模块。他只是把包 sound.effects 和它里面定义的所有内容导入进来（可能运行 `__init__.py` 里定义的初始化代码），并且他不会破坏掉我们在这句话之前导入的所有明确指定的模块，如：

```
import sound.effects.echo
import sound.effects.surround
from sound.effects import *
```

在执行 from...import 前，包 sound.effects 中的 echo 和 surround 模块都被导入到当前的命名空间中了。（当然如果定义了 `__all__` 就更没问题了）。
通常我们并不主张使用 \* 这种方法来导入模块，因为这种方法经常会导致代码的可读性降低。不过这样倒的确是可以省去不少敲键的功夫，而且一些模块都设计成了只能通过特定的方法导入。使用 from Package import specific_submodule 这种方法永远不会有错，这也是推荐的方法。
如果在结构中包是一个子包（比如这个例子中对于包 sound 来说），而你又想导入兄弟包（同级别的包）你就得使用导入绝对的路径来导入。比如，如果模块 sound.filters.vocoder 要使用包 sound.effects 中的模块 echo，你就要写成 from sound.effects import echo。

```
from . import echo
from .. import formats
from ..filters import equalizer
```

无论是隐式的还是显式的相对导入都是从当前模块开始的。主模块的名字永远是 `__main__`，一个 Python 应用程序的主模块，应当总是使用绝对路径引用。
包还提供一个额外的属性`__path__`。这是一个目录列表，里面每一个包含的目录都有为这个包服务的`__init__.py`，得在其他`__init__.py` 被执行前定义。可以修改这个变量，用来影响包含在包里面的模块和子包。
这个功能并不常用，一般用来扩展包里面的模块。

### 五、标准模块

Python 本身带着一些标准的模块库，有些模块直接被构建在解析器里，这些虽然不是一些语言内置的功能，但是他却能很高效的使用，甚至是系统级调用也没问题。这些组件会根据不同的操作系统进行不同形式的配置，比如 winreg 这个模块就只会提供给 Windows 系统。

#### 5.1、OS

```
os.environ          # 包含环境变量的映射
os.sep              # / ，路径中使用的分隔符
os.pathsep          # : ，分割不同路径的分隔符
os.linesep          # \n，行分隔符

os.urandom(n)       # b'H\x8b\xa9'，返回 n 个字节的强加密随机数据
os.getcwd()         # 返回当前所在的目录
os.chdir('/Users')  # /Users，修改当前所在目录
os.listdir()        # ['Shared', 'songsong.zhang']，返回当前目录下所有目录与文件
os.mkdir('hello')   # 文件夹创建，如果文件夹已存在则报错
or.rmdir('w')       # 删除文件夹，如果已经不存在则报错
os.rename('h', 'w') # 重命名，h 改为 w
d = '/Users/songsong.zhang/Desktop/a/c'
os.makedirs(d)      # 创建多层目录

os.path.exists('y') # 判断是否存在
os.path.isdir('py') # 判断是否是文件夹
os.path.isfile('py')# 判断是否是文件
f = '/Users/songsong.zhang/Desktop/event-loop.jpg'
os.path.dirname(f)  # 返回文件的目录路径，'/Users/songsong.zhang/Desktop'
os.path.split(f)    # 返回元组，第一个是目录路径，第二个是文件名，('/Users/songsong.zhang/Desktop', 'event-loop.jpg')
os.path.basename(f) # 返回文件名称，'event-loop.jpg'
os.path.splitext(f) # 返回元组获取文件后缀名，('/Users/songsong.zhang/Desktop/event-loop', '.jpg')
os.path.join('/User', 'a', 'b', 'c') # 返回组合路径，'/User/a/b/c'

```

PS：其他常用函数/变量

- platform.system()：判断平台，分别返回 'Linux', 'Darwin', 'Java', 'Windows'
- sys.argv：命令行参数，包括脚本名

#### 5.2、DateTime 模块

- DateTime 模块转换参数表

  - %A：星期的名称，如 Monday
  - %B：月份名：如 January
  - %m：用数字表示的月份（01-12）
  - %d：用数字表示月份中的一天（01-31）
  - %Y：四位的年份，如 2015
  - %y：两位的年份，如 15
  - %H：24 小时制的小时数（00-23）
  - %l：12 小时制的小时数（01-12）
  - %p：am 或 pm
  - %M：分钟数（00-59）
  - %S：秒数（00-59）

- 常见函数/变量

```
from datetime import datetime, date, time as time2, timedelta
import time

# datetime.now 返回系统的当前时间
now_time = datetime.now()

# 基本同 datetime.now（如果 now 的可选参数 tz 时区为 None 或未指定的话）
print(datetime.today())
print(now_time)
print(now_time.date()) # 获取日期
print(now_time.time()) # 获取时间
print(now_time.year, now_time.month, now_time.day) # 打印年、月、日属性
print(now_time.hour, now_time.minute, now_time.second, now_time.microsecond) # 打印时、分、秒、毫秒属性

# 获取从格林威治时间到现在的毫秒数
print(time.time())

# 自定义日期与事件
d = datetime(2020, 10, 30, 14, 5)
print(d)
d2 = date(2019, 2, 23)
print(d2)
t = time2(9, 2, 13)
print(t)

# 字符串转 datetime 对象
ds = '2018-10-3 13:42:09'
ds_t = datetime.strptime(ds, '%Y-%m-%d %H:%M:%S')
print(ds_t)
ds2 = '2018/10/3T13:42:09'
ds_t2 = datetime.strptime(ds2, '%Y/%m/%dT%H:%M:%S')
print(ds_t2)

# datetime 对象转字符串
n = datetime.now()
n_str = n.strftime('%Y/%m/%dT%H:%M:%S')
print(n_str)

# datetime 之间的加操作
n = datetime.now()
n_next = n + timedelta(days=5, hours=42)
print(n_next)

# datetime 之间的减操作
d1 = datetime(2018, 10, 15)
d2 = datetime(2018, 11, 13)
rest = d2 - d1
print(type(rest)) # <class 'datetime.timedelta'>
print(rest.days) # 29
```

### 六、第三方模块

- 安装网站：https://pypi.org
- 安装方法
  - pip install Django
  - python setup.py install

### 七、虚拟环境

- 定义

  - 建立在宿主环境上的独立容器
  - 具备和宿主环境相同的功能
  - 快速创建和删除、方便管理

- 优点

  - 独立：相互隔离、互不影响
  - 纯净：只有我一个项目用的包和依赖，好管理
  - 方便：摒弃频繁安装/卸载包和依赖

- virtualenv

```
# 创建虚拟环境
pip3 install virtualenv
virtualenv .pyenv

# active 后，在虚拟环境中进行安装
source ~/.pyenv/bin/activate
pip install xxx

# deactivate 退出虚拟环境
deactivate
```
