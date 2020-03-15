---
title: python 协程
categories:
  - note
---

python 的协程基础介绍

<!--more-->

### 一、协程简介

- 协程就是协同多任务
- 协程就是在一个进程或一个线程中执行
- 不需要锁机制
- 对多核 CPU 的利用——多进程+协程
- Python3.5 以前使用生成器（yield）来实现协程
- Python3.5 以后使用 async 和 await 关键字实现
  - async 关键字
    - 定义特殊函数
    - 当被调用时，不执行里面的代码，而是返回一个协程对象
    - 在事件循环中调度其执行前，协程对象不执行任何操作
  - await 关键字
    - 等待协程执行完成
    - 当遇到阻塞调用的函数的时候，使用 await 方法将协程的控制权让出，以便 loop 调用其他的协程
- asyncio 模块
  - get_event_loop 获得事件循环队列
  - run_until_complete() 注册任务到队列
  - 在事件循环中调度其执行前，协程对象不执行任何操作
  - asyncio 模块用于事件循环
- 协程之间的数据通信
  - 嵌套调用
  - 队列

### 二、yield 创建协程

```
def count_down(n):
    """ 倒计时效果 """
    while n > 0:
        yield n
        n -= 1

def yield_test():
    """ 实现协程函数 """
    while True:
        n = (yield)
        print(n)

if __name__ == '__main__':
    # rest = count_down(5)
    # print(next(rest))
    # print(next(rest))
    # print(next(rest))
    # print(next(rest))
    # print(next(rest))
    rest = yield_test()
    next(rest)
    rest.send('6666')
    rest.send('6666')
```

### 三、async 创建协程

```
import asyncio

async def do_sth(x):
    """ 定义协程函数 """
    print('等待中: {0}'.format(x))
    # 立即返回一个 Future 对象，下次循环的判断是否已经过 2s，而不是阻塞等待
    await asyncio.sleep(x)

# 判断是否为协程函数
print(asyncio.iscoroutinefunction(do_sth))

coroutine = do_sth(5)
# 事件的循环队列
loop = asyncio.get_event_loop()
# 注册任务
task = loop.create_task(coroutine)
print('1、task: ', task)
# 等待协程任务执行结束
loop.run_until_complete(task)
print('2、task: ', task)
```

### 四、协程嵌套调用通信

```
import asyncio

async def compute(x, y):
    print('计算 x + y => {0} + {1}'.format(x, y))
    # time.sleep 是同步阻塞接口
    await asyncio.sleep(3)
    return x + y

async def get_sum(x, y):
    rest = await compute(x, y)
    print('{0} + {1} = {2}'.format(x, y, rest))

# 拿到事件循环
loop = asyncio.get_event_loop()
loop.run_until_complete(get_sum(1, 2))
loop.close()
```

### 五、协程队列通信

```
# 1. 定义一个队列
# 2. 让两个协程来进行通信
# 3. 让其中一个协程往队列中写入数据
# 4. 让另一个协程从队列中删除数据
import asyncio
import random

async def add(store, name):
    """
    写入数据到队列
    :param store: 队列的对象
    :return:
    """
    for i in range(5):
        # 往队列中添加数字
        num = '{0} - {1}'.format(name, i)
        await asyncio.sleep(random.randint(1, 5))
        await store.put(i)
        print('{2} add one ... {0}, size: {1}'.format(
            num, store.qsize(), name))

async def reduce(store):
    """
    从队列中删除数据
    :param store:
    :return:
    """
    for i in range(10):
        rest = await store.get()
        print(' reduce one.. {0}, size: {1}'.format(rest, store.qsize()))

if __name__ == '__main__':
    # 准备一个队列
    store = asyncio.Queue(maxsize=5)
    a1 = add(store, 'a1')
    a2 = add(store, 'a2')
    r1 = reduce(store)

    # 添加到事件队列
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.gather(a1, a2, r1))
    loop.close()
```
