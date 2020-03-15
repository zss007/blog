---
title: python 进程
categories:
  - note
---

python 的进程基础介绍

<!--more-->

### 一、进程简介

- 一个执行中的程序
- 每个进程都拥有自己的地址空间、内存、数据栈以及其他用于跟踪执行的辅助数据
- 操作系统管理其上所有进程的执行，并为这些进程合理地分配时间
- 进程也可以通过派生（fork 或 spawn）新的进程来执行其他任务
- 进程的实现
  - 使用 multiprocessing 实现多进程代码
  - 用 multiprocessing.Process 创建进程
  - start() 启动进程
  - os.getpid() 获得进程的 ID
  - 进程间的通信：通过 Queue、Pipes 等实现进程之间的通信

### 二、进程的实现

```
import os
import time
from multiprocessing import Process

def do_sth(name):
    """
    进程要做的事情
    :param name: str 进程的名称
    """
    print('进程的名称：{0}， pid: {1}'.format(name, os.getpid()))
    time.sleep(15)
    print('进程要做的事情')

class MyProcess(Process):

    def __init__(self, name, *args, **kwargs):
        self.my_name = name
        # print(self.name)
        super().__init__(*args, **kwargs)
        print(self.name)

    def run(self):
        print('MyProcess进程的名称：{0}， pid: {1}'.format(
            self.my_name, os.getpid()))
        time.sleep(15)
        print('MyProcess进程要做的事情')


if __name__ == '__main__':
    # 函数形式创建进程
    # p = Process(target=do_sth, args=('my process', ))
    # class 形式创建进程
    p = MyProcess('my process class')
    # 启动进程
    p.start()
    # 挂起进程
    p.join()
```

### 三、进程之间的通信

```
from multiprocessing import Process, Queue, current_process
import random
import time

class WriteProcess(Process):
    """ 写的进程 """
    def __init__(self, q, *args, **kwargs):
        self.q = q
        super().__init__(*args, **kwargs)

    def run(self):
        """ 实现进程的业务逻辑 """
        # 要写的内容
        ls = [
            "第一行内容",
            "第2行内容",
            "第3行内容",
            "第4行内容",
        ]
        for line in ls:
            print('写入内容: {0} - {1}'.format(line, current_process().name))
            self.q.put(line)
            # 每写入一次，休息1-5秒
            time.sleep(random.randint(1, 5))

class ReadProcess(Process):
    """ 读取内容进程 """
    def __init__(self, q, *args, **kwargs):
        self.q = q
        super().__init__(*args, **kwargs)

    def run(self):
        while True:
            content = self.q.get()
            print('读取到的内容：{0} - {1}'.format(content, self.name))

if __name__ == '__main__':
    # 通过Queue共享数据
    q = Queue()
    # 写入内容的进程
    t_write = WriteProcess(q)
    t_write.start()
    # 读取进程启动
    t_read = ReadProcess(q)
    t_read.start()

    t_write.join()
    # t_read.join()

    # 因为读的进程是死循环，无法等待其结束，只能强制终止
    t_read.terminate()
```

### 四、多进程的锁

```
import random
from multiprocessing import Process, Lock, RLock

import time

class WriteProcess(Process):
    """ 写入文件 """

    def __init__(self, file_name, num, lock, *args, **kwargs):
        # 文件的名称
        self.file_name = file_name
        self.num = num
        # 锁对象
        self.lock = lock
        super().__init__(*args, **kwargs)

    def run(self):
        """ 写入文件的主要业务逻辑 """
        with self.lock:
        # try:
        #     # 添加锁
        #     self.lock.acquire()
        #     print('locked')
        #     self.lock.acquire()
        #     print('relocked')
            for i in range(5):
                content = '现在是： {0} : {1} - {2} \n'.format(
                    self.name,
                    self.pid,
                    self.num
                )
                with open(self.file_name, 'a+', encoding='utf-8') as f:
                    f.write(content)
                    time.sleep(random.randint(1, 5))
                    print(content)
        # finally:
        #     # 释放锁
        #     self.lock.release()
        #     self.lock.release()

if __name__ == '__main__':
    file_name = 'test.txt'
    # 所的对象
    lock = RLock()
    for x in range(5):
        p = WriteProcess(file_name, x, lock)
        p.start()
```

### 五、进程池

```
import random
import time
from multiprocessing import current_process, Pool

def run(file_name, num):
    """
    进程执行的业务逻辑
    往文件中写入数据
    :param file_name: str 文件名称
    :param num: int 写入的数字
    :return: str 写入的结果
    """
    with open(file_name, 'a+', encoding='utf-8') as f:
        # 当前的进程
        now_process = current_process()
        # 写入的内容
        content = '{0} - {1}- {2}'.format(
            now_process.name,
            now_process.pid,
            num
        )
        f.write(content)
        f.write('\n')
        # 写完之后随机休息1-5秒
        time.sleep(random.randint(1, 5))
        print(content)
    return 'ok'

if __name__ == '__main__':
    file_name = 'test_pool.txt'
    # 进程池
    pool = Pool(2)
    rest_list = []
    for i in range(20):
        # 同步添加任务
        # rest = pool.apply(run, args=(file_name, i))
        rest = pool.apply_async(run, args=(file_name, i))
        rest_list.append(rest)
        print('{0}--- {1}'.format(i, rest))
    # 关闭池子
    pool.close()
    pool.join()
    # 查看异步执行的结果
    print(rest_list[0].get())
```
