---
title: python 装饰器与异常处理
categories:
  - note
---

python 装饰器与异常处理

<!--more-->

### 一、装饰器

- 用于拓展原来函数功能
- 返回函数的函数
- 在不用更改原函数的代码前提下给函数增加新的功能

```
from functools import wraps

def log_simple(func):
    """ 记录函数执行的日志 """
    def wrapper():
        print('start...')
        func()
        print('end..')
    return wrapper

def log_simple2(func):
    """ 记录函数执行的日志 """
    def wrapper():
        print('开始进入...')
        func()
        print('结束..')
    return wrapper

def log(name=None):
    """ 记录函数执行的日志 """

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # *args：元祖；**kwargs：字典
            print('{0}.start...'.format(name))
            print(args)
            print(kwargs)
            rest = func(*args, **kwargs)
            print('{0}.end..'.format(name))
            return rest
        return wrapper
    return decorator

# 无参多个装饰器，注意执行顺序
@log_simple
@log_simple2
def hello():
    """ 简单功能模拟 """
    print('hello world')

# 有参
@log('from add')
def add(a, b, *args, **kwargs):
    """ 参数相加 """
    return a + b

def eat(cls):
    """ 吃东西装饰器 """
    cls.eat = lambda self: print('{0}>我要吃东西'.format(self.name))
    return cls

# 类装饰器
@eat
class Cat(object):
    """ 猫类 """

    def __init__(self, name):
        self.name = name

if __name__ == '__main__':
    # 无参
    hello()
    # start...
    # 开始进入...
    # hello world
    # 结束..
    # end..

    # 有参
    rest = add(5, 6, k=5, v=6)
    print(rest) # 11
    # from add.start...
    # (5, 6)
    # {'k': 5, 'v': 6}
    # from add.end..

    # 通过 @wraps(func) 来修复 __doc__ 和 __name__ 的值，否者使用 wrapper 的相应属性
    print('doc: {0}'.format(add.__doc__)) # doc:  参数相加
    print('name: {0}'.format(add.__name__)) # name: add

    # 类装饰器
    cat = Cat('小黑')
    cat.eat() # 小黑>我要吃东西
```

### 二、迭代器

- 迭代意味着重复多次，就像循环那样
- 实现了方法 `__iter__` 的对象是可迭代的，可使用 for 循环取值，而实现了方法 `__next__` 的对象是迭代器
- 调用方法 `__next__` 时（或 next()）, 迭代器返回其下一个值
- 如果迭代器没有可供返回的值，触发 StopIteration 异常
- 列表、字典、元组均为可迭代对象，所有可直接作用于 for...in 循环的数据类型都被称为可迭代对象
- 可使用 iter() 将可迭代对象转换为迭代器

```
# 通过可迭代对象调用内置函数 iter，可获得一个迭代器
l = [1, 2, 3]
l1 = iter(l)
print(next(l1))  # 1
print(next(l1))  # 2
print(next(l1))  # 3
print(next(l1))  # StopIteration error

# 自定义迭代器
class PowNumber(object):
    """
    迭代器
    生成1,2,3,4,5,... 数的平方
    """
    value = 0

    def __next__(self):
        self.value += 1
        if self.value > 10:
            raise StopIteration
        return self.value * self.value

    def __iter__(self):
        return self

if __name__ == '__main__':
    pow = PowNumber()
    # 调用 __next__ 方法
    print(pow.__next__()) # 1
    # 调用内置 next 方法
    print(next(pow)) # 4
    # 循环迭代器
    for i in pow:
        print(i, end=" ") # 9 16 25 36 49 64 81 100
    print('')
```

### 三、生成器

- 生成器是一种使用普通函数语法定义的迭代器
- 包含 yield 语句的函数都被称为生成器
- 不使用 return 返回一个值，而是可以生成多个值，每次一个
- 每次使用 yield 生成一个值后，函数都将冻结，即在此停止执行
- 被重新唤醒后，函数将从停止的地址开始继续执行

```
def pow():
    yield 1
    yield 2
    yield 3
    yield 4

def pow_number():
    # 通过推导式得到生成器
    return (x * x for x in [1, 2, 3, 4, 5])

def pow_number2():
    for x in [1, 2, 3, 4, 5]:
        yield x * x

if __name__ == '__main__':
    rest = pow()
    print(next(rest)) # 1
    for i in rest:
        print(i, end=" ") # 2 3 4
    print("")

    rest2 = pow_number2()
    print(rest2.__next__()) # 1
    print(next(rest2)) # 4

# 综合实例：模拟内置 range 函数
def use_range():
    """ python内置的range函数 """
    for i in range(5, 10):
        print(i, end=" ") # 5 6 7 8 9
    print("")

class IterRange(object):
    """ 使用迭代器来模拟range函数 """
    def __init__(self, start, end):
        self.start = start - 1
        self.end = end

    def __next__(self):
        self.start += 1
        if self.start >= self.end:
            raise StopIteration
        return self.start

    def __iter__(self):
        return self

class GenRange(object):
    """ 使用生成器来模拟range函数 """
    def __init__(self, start, end):
        self.start = start - 1
        self.end = end

    def get_num(self):
        while True:
            if self.start >= self.end - 1:
                break
            self.start += 1
            yield self.start

def get_num(start, end):
    start -= 1
    while True:
        if start >= end - 1:
            break
        start += 1
        yield start

if __name__ == '__main__':
    use_range()
    print('--------------------')
    iter = IterRange(5, 10)
    # print(next(iter))
    l = list(iter) # [5, 6, 7, 8, 9]
    print(l)

    print('--------------------')
    gen = GenRange(5, 10).get_num() # <generator object GenRange.get_num at 0x102750d68>
    print(gen) # [5, 6, 7, 8, 9]
    # print(next(gen))
    print(list(gen))

    print('--------------------')
    gen_f = get_num(5, 10)
    print(gen_f) # <generator object get_num at 0x102750de0>
    print(list(gen_f)) # [5, 6, 7, 8, 9]
```

### 四、异常处理

- 每个异常都是某个类的实例
- 发生了异常如果不捕获，则程序将终止执行
- 有一些内置的异常类
- 使用 try...except 捕获多个指定异常
- 可选的 else 子句，必须放在所有的 except 子句之后，将在 try 子句没有发生任何异常的时候执行
- 使用 raise 语句抛出一个指定的异常：raise [Exception [, args [, traceback]]]
- 只想知道是否抛出了一个异常，并不想去处理它，一个简单的 raise 语句就可以再次把它抛出

```
try:
    raise NameError('HiThere')
except NameError:
    print('An exception flew by!')
    raise
```

- finally 子句，无论是否发生异常都将执行最后的代码

```
try:
    raise KeyboardInterrupt
finally:
    print('Goodbye, world!')

# 先执行 finally 子句，再抛出异常（没有任何的 except 把它截住，那么这个异常会在 finally 子句执行后被抛出）
Goodbye, world!
Traceback (most recent call last):
  File "/test.py", line 2, in <module>
    raise KeyboardInterrupt
KeyboardInterrupt
```

#### 4.1、异常的捕获

```
def test_div(num1, num2):
    """ 当除数为0时 """
    return num1 / num2

def test_file():
    """ 读取文件 """
    try:
        f = open('testx.txt', 'r', encoding='utf-8')
        rest = f.read()
        print(rest)
    except Exception as err:
        print('open file error')
        print(err)
    finally:
        try:
            f.close()
            print('closed')
        except:
            pass

if __name__ == '__main__':
    try:
        rest = test_div(5, 0)
        print(rest)
    # 统一处理多个错误
    except (ZeroDivisionError, TypeError) as err:
        print('反正就是报错了')
        print(err)
    # except ZeroDivisionError:
    #     print('报错了，除数不能为0')
    # except TypeError:
    #     print('报错了，请输入数字')

    test_file()
```

#### 4.2、自定义异常

```
class ApiException(Exception):
    """ 我的自定义异常 """
    err_code = ''
    err_msg = ''

    def __init__(self, err_code=None, err_msg=None):
        """
        :param err_code:
        :param err_msg:
        """
        self.err_code = self.err_code if self.err_code else err_code
        self.err_msg = self.err_msg if self.err_msg else err_msg

    def __str__(self):
        return 'Error: {0} - {1}'.format(self.err_code, self.err_msg)

class BadPramsException(ApiException):
    """ 参数不正确 """
    err_code = '40002'
    err_msg = '两个参数必须都是整数'

def divide(num1, num2):
    """ 除法的实现 """
    # 两个数必须为整数
    if not isinstance(num1, int) or not isinstance(num2, int):
        raise BadPramsException()
    # 除数不能为0
    if num2 == 0:
        raise ApiException('400000', '除数不能为0')
    return num1 / num2

if __name__ == '__main__':

    try:
        rest = divide(5, 's')
        print(rest)
    # except BadPramsException as e:
    #     print('----------------')
    #     print(e)
    except ApiException as err:
        print('出错了')
        print(err)
```

#### 4.3、抛出异常及异常的传递

```
class MyException(Exception):
    """ 自定义异常类 """
    pass

def v_for():
    """ 自定义函数 """
    for i in range(1, 100):
        if i == 20:
            raise MyException
        print(i)

def call_v_for():
    """ 调用vfor函数 """
    print('开始调用v_for')
    try:
        v_for()
    except MyException:
        print('-------------------------')
    print('结束调用v_for')

def test_rasie():
    print('测试函数')
    call_v_for()
    print('测试完毕')

if __name__ == '__main__':
    test_rasie()
```
