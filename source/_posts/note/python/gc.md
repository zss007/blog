---
title: python 作用域与垃圾回收
categories:
  - note
---

python 作用域与垃圾回收

<!--more-->

### 一、命名空间

- 命名空间提供了在项目中避免名字冲突的一种方法
- 各个命名空间是独立的，没有任何关系的，所以一个命名空间中不能有重名，但不同的命名空间是可以重名而没有任何影响
- 三种命名空间：
  - 内置名称，Python 语言内置的名称，比如函数名 abs、char 和异常名称 BaseException、Exception 等等
  - 全局名称，模块中定义的名称，记录了模块的变量，包括函数、类、其它导入的模块、模块级的变量和常量
  - 局部名称，函数中定义的名称，记录了函数的变量，包括函数的参数和局部定义的变量（类中定义的也是）
- 命名空间查找顺序：局部的命名空间去 -> 全局命名空间 -> 内置命名空间
- 如果找不到变量 runoob，它将放弃查找并引发一个 NameError 异常
- 声明周期：取决于对象的作用域，如果对象执行完成，则该命名空间的生命周期就结束；故无法从外部命名空间访问内部命名空间的对象

```
# var1 是全局名称
var1 = 5
def some_func():
 
    # var2 是局部名称
    var2 = 6
    def some_inner_func():
 
        # var3 是内嵌的局部名称
        var3 = 7
```

### 二、作用域

- 作用域就是一个 Python 程序可以直接访问命名空间的正文区域
- 在一个 python 程序中，直接访问一个变量，会从内到外依次访问所有的作用域直到找到，否则会报未定义的错误
- 四种作用域：
  - 局部作用域（Local）：最内层，包含局部变量，比如一个函数/方法内部
  - 闭包函数外的函数（Enclosing）：包含了非局部(non-local)也非全局(non-global)的变量。比如两个嵌套函数，一个函数（或类） A 里面又包含了一个函数 B ，那么对于 B 中的名称来说 A 中的作用域就为 nonlocal
  - 全局作用域（Global）：当前脚本的最外层，比如当前模块的全局变量
  - 内建作用域（Built-in）：包含了内建的变量/关键字等，最后被搜索
- 当内部作用域想修改外部作用域的变量时，就要用到 global 和 nonlocal 关键字

```
# 修改外部作用域
num = 1
def fun1():
    global num # 需要使用 global 关键字声明
    print(num) # 1
    num = 123
    print(num) # 123
fun1()
print(num) # 123

# 修改嵌套作用域
def outer():
    num = 10
    def inner():
        nonlocal num # nonlocal关键字声明
        num = 100
        print(num) # 100
    inner()
    print(num) # 100
outer()

# 特殊情况，局部作用域引用错误（test 函数中的 a 使用的是局部，未定义，无法修改）
a = 10
def test():
    a = a + 1
    print(a)
test()
# Traceback (most recent call last):
#   File "test.py", line 7, in <module>
#     test()
#   File "test.py", line 5, in test
#     a = a + 1
# UnboundLocalError: local variable 'a' referenced before assignment
# 修改方法：
# 1、a = a + 1 改为 b = a + 1
# 2、通过函数参数传递，def test() 改为 def test(a)
```

### 三、赋值语句内存分析

- 通过 id() 方法访问内存地址
- == 表示值相等，is 表示内存引用地址相同

```
a = 1
b = 1
a is b # True，如果是较小数字或较短字符串，python 让变量共用内存地址

e = 999999999
f = 999999999
e == f # True，值相等
e is f # False，如果是较大数字或较长字符串，内存地址不共用

g = []
h = []
id(g) == id(h) # False，引用类型内存地址不共用

# 综合实例
def extend_list(val, l=[]):
    print('------------------')
    print(l, id(l))
    l.append(val)
    print(l, id(l))
    return l

list1 = extend_list(10)
list2 = extend_list(123, [])
list3 = extend_list('a')

print(list1) # [10, 'a']
print(list2) # [123]
print(list3) # [10, 'a']
print(list3 is list1) # True，list1 和 list3 共用同一个内存地址
```

### 四、垃圾回收机制

- 以引用计数为主，分代收集为辅
- 如果一个对象的引用数为 0，python 虚拟机就会回收这个对象的内存
- 引用计数的缺陷是循环引用的问题

```
class Cat(object):
    def __init__(self):
        print('对象产生了: {0}'.format(id(self)))

    # 对象被回收的时候被调用
    def __del__(self):
        print('对象删除了: {0}'.format(id(self)))

def f0():
    """ 自动回收内存 """
    while True:
        c1 = Cat()

def f1():
    """ 一直在被引用，不会被回收 """
    l = []
    while True:
        c1 = Cat()
        l.append(c1)
        print(len(l))

if __name__ == '__main__':
    f1()
```

### 五、内存管理机制

#### 5.1、引用计数

- 每个对象都有存有指向该对象的引用总数
- 查看某个对象的引用计数 sys.getrefcount()
- 可以使用 del 关键字删除某个引用

```
import sys

l = []
l2 = l
l3 = l
l5 = l3
print(sys.getrefcount(l))  # 5，对象 l 被引用的数量，方法执行时有个临时的引用，会比实际看到的多 1
del l2 # 删除某个引用
print(sys.getrefcount(l)) # 4

print('xxxxxxxxxxxxx')
i = 1
print(sys.getrefcount(i)) # 177，基于 python 内部的内存共享机制
a = i
print(sys.getrefcount(i)) # 178
```

#### 5.2、垃圾回收

- 满足特定条件，自动启动垃圾回收
- 当 python 运行时，会记录其中分配对象（object allocation）和取消分配对象（object deallocation）的次数
- 当两者的差值高于某个阀值时，垃圾回收才会启动
- 查看阀值 gc.get_threshold()，返回 (threshold0, threshold1, threshold2)，比如 (700, 10, 10)
- 分代回收
  - python 将所有的对象分为 0，1，2 三代
  - 所有的新建对象都是 0 代对象
  - 当某一代对象经历过垃圾回收，依然存活，那么他将被归入下一代对象
- 手动回收 gc.collect([generation])，可指定具体哪一代进行回收，不传则回收所有
- objgraph 模块中的 count() 记录当前类产生的实例对象的个数

```
import gc
import sys
import objgraph

print(gc.get_threshold()) # (700, 10, 10)

class Persion(object):
    pass

class Cat(object):
    pass

p = Persion()
c = Cat()
p.name = 'Susan'

# 循环引用
p.pet = c
c.master = p
print(sys.getrefcount(p)) # 3
print(sys.getrefcount(c)) # 3

# 调用 del 解除循环引用状态
del p
del c

# 手动执行垃圾回收
gc.collect()

# 就算进行手动垃圾回收，引用次数仍为 1，因为存在循环引用，除非调用 del 方法
print(objgraph.count('Persion')) # 0
print(objgraph.count('Cat')) # 0
```

#### 5.3、内存管理机制

- 内存池机制
  - 当创建大量消耗小内存的对象时，频繁调用 new/malloc 会导致大量的内存碎片，致使效率降低
  - 内存池就是预先在内存中申请一定数量的，大小相等的内存块留作备用
  - 当有新的内存需求时，就先从内存池中分配内存给这个需求，不够了之后再申请新的内存
  - 这样做最显著的优势就是能够减少内存碎片，提升效率
- python3 内存管理机制 Pymalloc
  - 针对小对象（<=512bytes），pymalloc 会在内存池中申请内存空间
  - 当 >512bytes，则会 PyMem_RawMalloc() 和 PyMem_RawRealloc() 来申请新的内存空间