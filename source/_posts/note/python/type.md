---
title: python 数据类型
categories:
  - note
---

python 数据类型

### 一、基本数据类型

- Python 中的变量不需要声明。每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建
- 在 Python 中，变量就是变量，它没有类型，我们所说的"类型"是变量所指的内存中对象的类型
- 等号（=）用来给变量赋值
- 等号（=）运算符左边是一个变量名,等号（=）运算符右边是存储在变量中的值

```
# 允许同时为多个变量赋值
a = b = c = 1

# 可以为多个对象指定多个变量
a, b, c = 1, 2, "runoob"
```

- 标准数据类型
  - 六个标准的数据类型
    - Number
    - String
    - List
    - Dictionary
    - Tuple
    - Set
  - 是否可变
    - 不可变数据：Number（数字）、String（字符串）、Tuple（元组）
    - 可变数据：List（列表）、Dictionary（字典）、Set（集合）

### 二、Number

- 整数(int)
- 浮点型(float)
- 布尔型(bool)，值是 1（True） 和 0（False），它们可以和数字相加
- 复数(complex)，复数是一个数学概念，包含实部和虚部，如 1 + 2j、 1.1 + 2.2j，其中 1、1.1 为实部，2、2.2 为虚部
- 常用函数
  - abs 绝对值
  - max(x1, x2,...)
  - min(x1, x2,...)
  - pow(x, y)
  - round(x, [n]) 返回浮点数 x 的四舍五入值，如给出 n 值，则代表舍入到小数点后的位数
  - sqrt(x) 数字 x 的平方根

```
a, b, c, d = 20, 5.5, True, 4+3j
# <class 'int'> <class 'float'> <class 'bool'> <class 'complex'>
print(type(a), type(b), type(c), type(d))

a = 111
isinstance(a, int)

# isinstance 和 type 的区别：type() 不会认为子类是一种父类类型；isinstance() 会认为子类是一种父类类型
class A:
    pass

class B(A):
    pass

isinstance(A(), A)  # True
type(A()) == A  # True
isinstance(B(), A)  # True
type(B()) == A  # False

# 1234.57 str
format(1234.567, '0.2f')

# 1,234.567 str
format(1234.567, ',')

# 123,456,789.00 str
format(123456789, '0,.2f')

# 请您向8810381账户转账￥12,334.100 如遇格式化输出数字，则在 {} 增加 : 前缀，之后写上数字格式化语句
'请您向{}账户转账￥{:0,.3f}'.format('8810381', 12334.1)
```

### 三、String

- 拼接

```
# MF123
'MF' + '123'
'MF' + str(123)
```

- 反斜杠

```
# 使用反斜杠(\)转义特殊字符
print('Ru\noob')
# Ru
# oob

# 如果不想让反斜杠发生转义，可以在字符串前面添加一个 r，表示原始字符串
print(r'Ru\noob')
# Ru\noob

# 反斜杠(\)可以作为续行符，表示下一行是上一行的延续。也可以使用 """...""" 或者 '''...''' 跨越多行
```

- 大小写转换

  - str.lower() 转换为小写
  - str.upper() 转换为大写
  - str.capitalize() 字符串首字母大写
  - str.title() 每个单词首字母大写
  - str.swapcase() 大小写互换

- 格式化

```
# I love you
'{} {} you'.format('I', 'love')

# www.imooc.com
'{2}.{1}.{0}'.format('com', 'imooc', 'www')

# 我叫小明,我在3-2
print('我叫{p1},我在{p2}'.format(p1='小明', p2='3-2'))

# 在 : 后传入一个整数, 可以保证该域至少有这么多的宽度，用于美化表格时很有用
table = {'Google': 1, 'Runoob': 2, 'Taobao': 3}
for name, number in table.items():
    print('{0:10} ==> {1:10}'.format(name, number))
# Google     ==>          1
# Runoob     ==>          2
# Taobao     ==>          3

# 字典输出格式化
table = {'Google': 1, 'Runoob': 2, 'Taobao': 3}
print('Runoob: {Runoob:d}; Google: {Google:d}; Taobao: {Taobao:d}'.format(**table))
```

- 早期的格式化输出
  早期的字符串格式化使用 %s、%d、%f 来格式化字符串

```
# 我叫ben,今年25岁,体重70.30公斤
'我叫%s,今年%d岁,体重%.2f公斤'%('ben', 25, 70.3)
```

- 截取，语法：变量[头下标:尾下标:步长]

```
mystr = 'Runoob'

# 输出字符串第一个字符，R
print(mystr[0])

# 输出从第三个开始到第五个的字符，noo
print(mystr[2:5])

# 输出第一个到倒数第二个的字符，步长为 2，Rno
print(mystr[0:-1:2])

# 输出第三个到最后一个，noob
print(mystr[2:])

# 输出字符串两次，也可以写成 print(2 * str)，RunoobRunoob
print(mystr * 2)

# Python 字符串不能被改变。向一个索引位置赋值，比如 mystr[0] = 'm'会导致错误
```

- str.find(目标串, [开始位置], [结束位置])

```
# 9
'Nice to meet you'.find('ee')

# 21
'Nice to meet you, i need you help!'.find('ee', 17)

# -1
'Nice to meet you, i need you help!'.find('ee', 17, 22)
```

- rfind
  类似于 find，只不过是从右面开始查找，返回的是第一个字符的索引，如果查询不到同样返回 -1

- index
  跟 find() 方法一样，只不过如果 str 不在 mystr 中会报错，会报一个异常

- rindex
  类似 index，只不过是从右面开始查找，返回的是开始查询的索引，如果查询不到则报错

- str.count(mystr, start=0, end=len(str))
  返回 mystr 在 start 和 end 之间在 mystr 中出现的次数；如果找到需要的数据，返回出现的次数，如果没有找到，则返回 0

- endswith(suffix, beg=0, end=len(string))
  检查字符串是否以 obj 结束，如果 beg 或者 end 指定则检查指定的范围内是否以 obj 结束，如果是，返回 True,否则返回 False

- startswith(substr, beg=0,end=len(string))
  检查字符串是否是以指定子字符串 substr 开头，是则返回 True，否则返回 False。如果 beg 和 end 指定值，则在指定范围内检查

- isdigit()
  如果字符串只包含数字则返回 True 否则返回 False

- isnumeric()
  如果字符串中只包含数字字符，则返回 True，否则返回 False

- split(str="", num=string.count(str))

```
str = "this is string example....wow!!!"
# ['this', 'is', 'string', 'example....wow!!!']
print (str.split( ))       # 以空格为分隔符

# ['th', 's is string example....wow!!!']
print (str.split('i',1))   # 以 i 为分隔符，num 为分割次数，默认为 -1, 即分隔所有

# ['this is string example....', 'o', '!!!']
print (str.split('w'))     # 以 w 为分隔符
```

- join(seq)
  以指定字符串作为分隔符，将 seq 中所有的元素(的字符串表示)合并为一个新的字符串

```
# r,u,n,o,o,b
','.join(["r", "u", "n", "o", "o", "b"])
```

- in

```
# True
'ee' in 'Nice to meet you'
```

- str.replace(原始串, 目标串, [替换次数])

```
# aaaddbccc
'aaabbbccc'.replace('b', 'd', 2)

# aaadddccc
'aaabbbccc'.replace('b', 'd')
```

- 删除空白
  - str.lstrip() 删除左侧空白
  - str.rstrip() 删除右侧空白
  - str.strip() 删除两端空白

```
# Marry
'    Marry    '.strip(' ')
'####Marry####'.strip('#')
```

- 字符长度 len(str)
- 行分割，返回一个包含各个元素的列表
  - mystr.splitlines()

### 四、列表 List

```
# 创建
list1 = ['Google', 'Runoob', 1997, 2000]
list2 = [12, 3, 14]

# 访问 ['Runoob', 2000]
list1[1:4:2]

# 列表截取
inputWords = 'I like runoob'.split(" ")
inputWords=inputWords[-1::-1] # 三个参数分别为：最后一个元素；移动到列表末尾；逆向
' '.join(inputWords) # runoob like I

# ['Google', 'Runoob', 2001, 2000]
list1[2] = 2001

# ['Google', 'Runoob', 2000]
del list1[2]

# 3
len(list1)

# [1, 2, 3, 4, 5, 6]
[1, 2, 3] + [4, 5, 6]

# ['Hi!', 'Hi!', 'Hi!', 'Hi!']
['Hi!'] * 4

# True
3 in [1, 2, 3]

# 1 2 3
for x in [1, 2, 3]: print(x, end=" ")

# 14
max(list2)

# 3
min(list2)

# 将元组转换为列表
list(seq)

# 在列表末尾添加新的对象 [12, 3, 14, 13]
list2.append(13)

# 统计某个元素在列表中出现的次数 1
list2.count(3)

# 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表），可以是列表、元组、集合、字典
list.extend(seq)

# 从列表中找出某个值第一个匹配项的索引位置
list.index(obj)

# 将对象插入列表
list.insert(index, obj)

# 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
list.pop([index=-1])

# 移除列表中某个值的第一个匹配项
list.remove(obj)

# 反向列表中元素
list.reverse()

# 对原列表进行排序，key 指定可迭代对象中的一个元素来进行排序
list.sort( key=None, reverse=False)
# 降序 ['u', 'o', 'i', 'e', 'a']
vowels = ['e', 'a', 'u', 'o', 'i']
vowels.sort(reverse=True)
# 指定列表中的元素排序 [(4, 1), (2, 2), (1, 3), (3, 4)]
def takeSecond(elem):
    return elem[1]
random = [(2, 2), (3, 4), (4, 1), (1, 3)]
random.sort(key=takeSecond)

# 清空列表
list.clear()

# 复制列表，浅拷贝
list.copy()

# 推导式，语法：[被追加的数据 循环语句 循环或判断]
[i * 10 for i in range(10, 20)] # [100, 110, 120, 130, 140, 150, 160, 170, 180, 190]
[i * 10 for i in range(10, 20) if i % 2 == 0] # [100, 120, 140, 160, 180]

# 索引位置和对应值可以使用 enumerate() 函数同时得到
for i, v in enumerate(['tic', 'tac', 'toe']):
    print(i, v)

# 同时遍历两个或更多的序列，可以使用 zip() 组合
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
    print('What is your {0}?  It is {1}.'.format(q, a))

# 反向遍历一个序列，首先指定这个序列，然后调用 reversed() 函数
for i in reversed(range(1, 10, 2)):
    print(i) # 9 7 5 3 1

# 按顺序遍历一个序列，使用 sorted() 函数返回一个已排序的序列，并不修改原值
basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
for f in sorted(set(basket)):
    print(f) # apple banana orange pear
```

### 五、字典 Dict

```
# 创建 键必须是不可变的，如字符串，数字或元组；不允许同一个键出现两次，创建时如果同一个键被赋值两次，后一个值会被记住
dict1 = {'Name': 'Runoob', 'Age': 7, 'Class': 'First'}
dict2 = {'Alice': '2341', 'Beth': '9102', 'Cecil': 3258, , 98.6: 37}
# {'name': 'ben', 'age': 12}
dict2(name='ben', age=12)
# {'name': 10, 'age': 10, 'sex': 10}
dict4 = dict.fromkeys(('name', 'age', 'sex'), 10)

# 修改字典 {'Alice': '2341', 'Beth': '9102', 'Cecil': 8, 98.6: 37}
dict2['Cecil'] = 8
dict2[98.6] = 8

# 删除 {'Age': 7, 'Class': 'First'}
del dict1['Name']
# {}
dict1.clear()

# 计算字典元素个数，即键的总数
len(dict2)

# 输出字典，以可打印的字符串表示
str(dict2)

# 返回输入的变量类型，如果变量是字典就返回字典类型
type(dict2)

# 返回一个字典的浅复制
radiansdict.copy()

# 创建一个新字典，以序列seq中元素做字典的键，val为字典所有键对应的初始值
radiansdict.fromkeys()
seq = ('name', 'age', 'sex')
# {'age': 10, 'name': 10, 'sex': 10}, 第二个参数可选，默认为 None
dict3 = dict.fromkeys(seq, 10)

# 返回指定键的值，如果值不在字典中返回 default 值
radiansdict.get(key, default=None)

# 如果键在字典 dict 里返回 true，否则返回 false
key in dict

# 以列表返回可遍历的(键, 值) 元组数组
radiansdict.items()
# dict_items([('Name', 'Runoob'), ('Age', 7)])
{'Name': 'Runoob', 'Age': 7}.items()

# 返回一个迭代器，可以使用 list() 来转换为列表
radiansdict.keys()

# 返回一个迭代器，可以使用 list() 来转换为列表
radiansdict.values()

# 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default，并返回 default
radiansdict.setdefault(key, default=None)

# 返回并删除字典中的最后一对键和值，如果字典已经为空，则报 KeyError 异常
popitem()

# 删除字典给定键 key 所对应的值，返回值为被删除的值。key值必须给出。否则，返回default值。
pop(key[,default])

# 把字典dict2的键/值对更新到dict里
radiansdict.update(dict2)

# 生成散列值，多次运行时，每次结果不同
hash('ben') # 2712347711650408443
hash('ben') == hash('ben') # True，每次生成的结果都相同
hash(8838183) # 8838183，数字相同

# 字典生成式，语法：{key: value 循环语句 循环或判断}
{i: i * 10 for i in range(10, 20)} # {10: 100, 11: 110, 12: 120, 13: 130, 14: 140, 15: 150, 16: 160, 17: 170, 18: 180, 19: 190}

# 直接从键值对序列中构建字典
dict([('Runoob', 1), ('Google', 2), ('Taobao', 3)]) # {'Taobao': 3, 'Runoob': 1, 'Google': 2}

# 使用关键字参数指定键值对
dict(Runoob=1, Google=2, Taobao=3) # {'Runoob': 1, 'Google': 2, 'Taobao': 3}
```

### 六、元组 Tuple

- 元组与列表类似，不同之处在于元组的元素不能修改；
- 元组使用小括号，列表使用方括号；
- 元组创建很简单，只需要在括号中添加元素，并使用逗号隔开即可

```
# 创建
tup1 = ('Google', 'Runoob', 1997, 2000)
tup2 = () # 空元组
tup3 = "a", "b", "c", "d"   # 不需要括号也可以

# 访问
tup1[1:3] # ('Runoob', 1997)
tup1[-2] # 1997
tup1[1:] # ('Runoob', 1997, 2000)

# ('Google', 'Runoob', 1997, 2000, 'a', 'b', 'c', 'd') 修改元组（元组中的元素值是不允许修改的，但我们可以对元组进行连接组合）
tup1 + tup3

# 删除元组（元组被删除后，输出变量会有异常信息）
tup4 = ('Google', 'Runoob', 1997, 2000)
del tup4

# 元组运算符
(1, 2, 3) + (4, 5, 6) # (1, 2, 3, 4, 5, 6)，连接
('Hi!',) * 4 # ('Hi!', 'Hi!', 'Hi!', 'Hi!')，复制
3 in (1, 2, 3) # True，元素是否存在
for x in (1, 2, 3): print (x,) # 1 2 3，迭代

# 内置函数
len(tuple) # 计算元素个数
max(tuple) # 返回元组中元素最大值
min(tuple) # 返回元组中元素最小值
tuple(iterable) # 将可迭代系列转换为元组

# 不可变，指向的内存中的内容不可变
tup5 = ('r', 'u', 'n', 'o', 'o', 'b')
id(tup5) # 4461192104
tup5 = (1, 2, 3)
id(tup5) # 4461390080

# 如果元组内持有列表，那么列表的内容是允许被修改的
tup6 = ([1, 2, 3], [6, 8, 10])
item = tup6[0]
item[1] = 12
tup6 # ([1, 12, 3], [6, 8, 10])

# 如果元组只有一个元素时，必须后加 ,
(10) * 5 # 50
(10,) * 5 # (10, 10, 10, 10, 10)

# 序列类型相互转换（序列包含常用的数据结构：字符串 str、列表 list、元组 tuple、数字序列 Range）
- list() 转换为列表
- tuple() 转换为元组
- str() 用于将单个数据转为字符串
- join() 对列表进行拼接，要求列表中所有元素都是字符串
```

### 七、集合 Set

集合（set）是一个无序的不重复元素序列，可以使用大括号 { } 或者 set() 函数创建集合，注意：创建一个空集合必须用 set() 而不是 { }，因为 { } 是用来创建一个空字典。

```
# 创建，参数可以是字典、列表、元素、字符串、集合
set(['a', 'b']) # 从列表中转变
set('dajksd') # 从字符串中转变

# 去重
basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'} # {'orange', 'banana', 'pear', 'apple'}

# 快速判断元素是否在集合内
'orange' in basket # false

# 添加元素
basket.add('xxx') # 将 xxx 添加到集合中
basket.update(x) # 参数可以是列表，元组，字典
thisset = set(("Google", "Runoob", "Taobao"))
thisset.update({1,3}) # {1, 3, 'Google', 'Taobao', 'Runoob'}
thisset.update([1,4],[5,6], {'a': 'c'}, ('e', 'f')) # {1, 3, 4, 5, 6, 'a', 'e', 'f', 'Google', 'Taobao', 'Runoob'}

# 移除元素
s.remove(x) # 不存在会发生错误
s.discard(x) # 元素不存在，不会发生错误
s.pop() # 随机删除集合中的一个元素

# 计算集合元素个数
len(s)

# 清空集合
s.clear()

# 集合间的运算
a = set('abracadabra') # {'a', 'r', 'b', 'c', 'd'}
b = set('alacazam') # {'m', 'z', 'a', 'l', 'c'}
# 集合a中包含而集合b中不包含的元素
a - b # {'r', 'd', 'b'}
# 集合a或b中包含的所有元素
a | b # {'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
# 集合a和b中都包含了的元素
a & b # {'a', 'c'}
# 不同时包含于a和b的元素
a ^ b # {'r', 'd', 'b', 'm', 'z', 'l'}
# 判断两个集合的元素是否完全相同
a = b

# 常用方法
- copy() 拷贝一个集合
- isdisjoint() 判断两个集合是否包含相同的元素，如果没有返回 True，否则返回 False
- issubset() 判断指定集合是否为该方法参数集合的子集
- issuperset() 判断该方法的参数集合是否为指定集合的子集
- difference() 返回多个集合的差集
- difference_update() 移除集合中的元素，该元素在指定的集合也存在，更新原有集合
- intersection() 返回集合的交集
- intersection_update() 返回集合的交集，更新原有集合
- symmetric_difference 返回两个集合中不重复的元素集合
- symmetric_difference_update() 移除当前集合中在另外一个指定集合相同的元素，并将另外一个指定集合中不同的元素插入到当前集合中
- union() 返回两个集合的并集

# 集合生成式，语法：{被追加的数据 循环语句 循环或判断}
{i * 10 for i in range(10, 20) if i % 2 == 0} # {160, 100, 140, 180, 120}
```

### 八、类型转换

- 字符串转数字：int(字符串) float(字符串) complex(x, y)

```
# 报错
int('101.5')

# 101
int(float('101.5'))

# complex(x, y) 将 x 和 y 转换到一个复数，实数部分为 x，虚数部分为 y。x 和 y 是数字表达式。
```

- 数字转字符串：str(数字)
- 数字与布尔：数字 0 代表 False，非 0 代表 True
- 常用方法
  - int(x [,base]) 将 x 转换为一个整数
  - float(x) 将 x 转换到一个浮点数
  - complex(real [,imag]) 创建一个复数
  - str(x) 将对象 x 转换为字符串
  - tuple(s) 将序列 s 转换为一个元组
  - list(s) 将序列 s 转换为一个列表
  - set(s) 转换为可变集合
  - dict(d) 创建一个字典。d 必须是一个 (key, value)元组序列
  - frozenset(s) 转换为不可变集合
  - hex(x) 将一个整数转换为一个十六进制字符串
  - oct(x) 将一个整数转换为一个八进制字符串
