---
title: python 文件
categories:
  - note
---

python 文件

<!--more-->

### 一、文件读写模式

- r：读取模式（默认值）
- w：写入模式，覆盖原内容
- x：独占写入模式
- a：附加模式，追加在原内容后
- b：二进制模式（与其他模式结合使用）
- t：文本模式（默认模式，与其他模式结合使用）
- +：读写模式（与其他模式结合使用）

| 模式       | r   | r+  | w   | w+  | a   | a+  |
| ---------- | --- | --- | --- | --- | --- | --- |
| 读         | +   | +   |     | +   |     | +   |
| 写         |     | +   | +   | +   | +   | +   |
| 创建       |     |     | +   | +   | +   | +   |
| 覆盖       |     |     |     | +   | +   |     |
| 指针在开始 | +   | +   | +   | +   |     |     |
| 指针在结尾 |     |     |     |     | +   | +   |

### 二、文件的打开和关闭

```
# 使用 open 函数打开文件（如果文件不存在则新建一个文件）
f = open('test.py') # <_io.TextIOWrapper name='test.py' mode='r' encoding='UTF-8'>

# 使用 close 函数关闭文件
f.close() # 要做异常处理

# 使用 with 语法（到达该语句末尾时，将自动关闭文件，即便出现异常亦如此）
with open('test.py') as f:
    do_something(f)
```

### 三、文件的读写

- read()：读取文件，可以指定参数，表示读几个字符（字节），如果没传参数或者参数为负, 那么读取文件所有内容并返回

```
def read_file():
    f = open('file.md', encoding='utf-8')
    print(f.read(8)) # 先读取 8 个字符
    rest = f.read() # 再前面的基础上读取剩余字符
    print(rest)
    f.close()

read_file()
```

- readline()：读取一行数据（可以指定参数，表示读前几个字符，指定参数效果同 read 函数；如果返回为空字符串，则说明读取到最后一行）

```
def read_file():
    f = open('file.md', encoding='utf-8')
    rest = f.readline()
    print(rest)
    f.close()

read_file()
```

- readlines()：读取所有行，并返回列表（可设置参数 sizehint, 读取指定长度的字节, 并且将这些字节按行分割）
- seek(offset[, whence])
  - offset：开始的偏移量，也就是代表需要移动偏移的字节数，如果是负数表示从倒数第几位开始
  - whence：可选，默认值为 0。给 offset 定义一个参数，表示要从哪个位置开始偏移；0 代表从文件开头开始算起，1 代表从当前位置开始算起，2 代表从文件末尾算起

### 四、文件写入

- 使用 write 函数向打开的文件对象写入内容

```
def write_file():
    f = open('test.txt', 'w', encoding='utf-8')
    f.write('hello\n')
    f.write('world')
    f.close()

write_file()
```

- 使用 writelines 函数向打开的文件对象写入多行内容

```
def write_file():
    f = open('test.txt', 'w', encoding='utf-8')
    l = ['第一行\n', '第二行']
    f.writelines(l)
    f.close()

write_file()

# 追加在原内容后
def write_log():
    f = open('test.txt', 'a', encoding='utf-8')
    l = ['第一行\n', '第二行\n']
    f.writelines(l)
    f.close()

write_log()
write_log()

# 读写模式
def read_and_write():
    f = open('test.txt', 'r+', encoding='utf-8')
    read_test = f.read()
    if '1' in read_test:
        f.write('bbb')
    else:
        f.write('aaa')
    f.close()

read_and_write()
```

### 五、数据备份

- pickle 模块实现了基本的数据序列和反序列化
- 通过 pickle 模块的序列化操作我们能够将程序中运行的对象信息保存到文件中去，永久存储
- 通过 pickle 模块的反序列化操作，我们能够从文件中创建上一次程序保存的对象
- 基本语法：
  - pickle.dump(obj, file, [,protocol])
  - protocol 为序列化使用的协议版本：0：ASCII 协议；1：老式的二进制协议；2：2.3 版本引入的新二进制协议，较以前的更高效

```
import pprint, pickle

# 使用 pickle 模块将数据对象保存到文件
data_w = {
    'a': [1, 2.0, 3, 4+6j],
    'b': ('string', u'Unicode string'),
    'c': None
}

list_w = [1, 2, 3]
list_w.append(list_w)

file_w = open('test.pkl', 'wb')
pickle.dump(data_w, file_w)
# Pickle the list using the highest protocol available.
pickle.dump(list_w, file_w, -1)
file_w.close()

file_r = open('test.pkl', 'rb')
data1 = pickle.load(file_r)
pprint.pprint(data1)
data2 = pickle.load(file_r)
pprint.pprint(data2)
file_r.close()
```
