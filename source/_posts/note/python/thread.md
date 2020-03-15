---
title: python 线程
categories:
  - note
---

python 的线程基础介绍

<!--more-->

### 一、线程简介

- 在同一个进程下执行，并共享相同的上下文
- 一个进程中的各个线程与主线程共享同一片数据空间
- 线程包括开始、执行顺序和结束三部分
- 它可以被抢占（中断）和临时挂起（也称为睡眠）——线程让步
- 一般是以并发方式执行
- 线程的实现

  - 用 threading 模块代替 thread 模块
  - 用 threading.Tread 创建线程
  - start() 启用线程
  - join() 挂起线程

- threading 模块的对象

| 对象             | 描述                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------ |
| Thread           | 表示一个执行线程的对象                                                               |
| Lock             | 锁原语对象（和 thread 模块中的锁一样）                                               |
| RLock            | 可重入锁对象，使单一线程可以（再次）获得已持有的锁（递归锁）                         |
| Condition        | 条件变量对象，使得一个线程等待另一个线程满足特定的“条件”，比如改变状态或某个数据值   |
| Event            | 条件变量的通用版本，任意数量的线程等待某个事件的发生，在该事件发生后所有线程将被激活 |
| Semaphore        | 为线程间共享的有限资源提供了一个“计数器”，如果没有可用资源时会被阻塞                 |
| BoundedSemaphore | 与 Semaphore 相似，不过它不允许超过初始值                                            |
| Timer            | 与 Thread 相似，不过它要在运行前等待一段时间                                         |
| Barrier          | 创建一个“障碍”，必须达到指定数量的线程后才可以继续                                   |

- Thread 对象数据属性

| 属性   | 描述                                 |
| ------ | ------------------------------------ |
| name   | 表示一个执行线程的对象               |
| ident  | 线程的标识符                         |
| daemon | 布尔标志，表示这个线程是否是守护线程 |

- Thread 对象方法

| 属性                | 描述                                                                     |
| ------------------- | ------------------------------------------------------------------------ |
| `_init_`()            | 实例化一个线程对象，需要有一个可调用的 target，以及其参数 args 或 kwargs |
| start()             | 开始执行该线程                                                           |
| run()               | 定义线程功能的方法（通常在子类中被应用开发者重写）                       |
| join(timeout=None)  | 直至启动的线程终止之前一直挂起；除非给出了 timeout（秒），否则一直阻塞   |
| getName()           | 返回线程名                                                               |
| setName(name)       | 设置线程名                                                               |
| isAlivel/is_alive() | 布尔标志，表示这个线程是否还存活                                         |
| isDaemon()          | 如果是守护线程，则返回 True；否则返回 False                              |
| setDaemon()         | 把线程的守护标志设定为布尔值 daemonic（必须在线程 start()之前调用）      |

### 二、线程的实现

```
import threading
import time

def loop():
    """ 新的线程执行的代码 """
    n = 0
    while n < 5:
        print(n)
        now_thread = threading.current_thread()
        print('[loop]now  thread name : {0}'.format(now_thread.name))
        time.sleep(1)
        n += 1

class LoopThread(threading.Thread):
    """ 自定义线程 """
    n = 0

    def run(self):
        while self.n < 5:
            print(self.n)
            now_thread = threading.current_thread()
            print('[loop]now  thread name : {0}'.format(now_thread.name))
            time.sleep(1)
            self.n += 1

def use_thread():
    """ 使用线程来实现 """
    # 当前正在执行的线程名称
    now_thread = threading.current_thread()
    print('now  thread name : {0}'.format(now_thread.name))
    # 函数方法设置线程
    # t = threading.Thread(target=loop, name='loop_thread')
    # class 方法设置线程
    t = LoopThread(name='loop_thread_oop')
    # 启动线程
    t.start()
    # 挂起线程
    t.join()

if __name__ == '__main__':
    use_thread()
```

### 三、多线程的锁

```
import threading
import time

# 获得一把锁
my_lock = threading.Lock()
your_lock = threading.RLock()

# 我的银行账户
balance = 0

def change_it(n):
    """ 改变我的余额 """
    global balance

    # 方式一，使用with
    with your_lock:
        balance = balance + n
        time.sleep(2)
        balance = balance - n
        time.sleep(1)
        print('-N---> {0}; balance: {1}'.format(n, balance))

    # 方式二
    # try:
    #     print('start lock')
    #     # 添加锁
    #     your_lock.acquire()
    #     print('locked one ')
    #     # 资源已经被锁住了，不能重复锁定, 产生死锁
    #     your_lock.acquire()
    #     print('locked two')
    #     balance = balance + n
    #     time.sleep(2)
    #     balance = balance - n
    #     time.sleep(1)
    #     print('-N---> {0}; balance: {1}'.format(n, balance))
    # finally:
    #     # 释放掉锁
    #     your_lock.release()
    #     your_lock.release()

class ChangeBalanceThread(threading.Thread):
    """
    改变银行余额的线程
    """
    def __init__(self, num, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.num = num

    def run(self):
        for i in range(100):
            change_it(self.num)

if __name__ == '__main__':
    t1 = ChangeBalanceThread(5)
    t2 = ChangeBalanceThread(8)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    print('the last: {0}'.format(balance))
```

### 四、线程池

```
import time
import threading
# ThreadPoolExecutor 性能比 Pool 性能更强
from concurrent.futures import ThreadPoolExecutor
from multiprocessing.dummy import Pool

def run(n):
    """ 线程要做的事情 """
    time.sleep(2)
    print(threading.current_thread().name, n)

def main():
    """ 使用传统的方法来做任务 """
    t1 = time.time()
    for n in range(100):
        run(n)
    print(time.time() - t1)

def main_use_thread():
    """ 使用线程优化任务 """
    # 资源有限，最多只能跑10个线程
    t1 = time.time()
    ls = []
    for count in range(10):
        for i in range(10):
            t = threading.Thread(target=run, args=(i,))
            ls.append(t)
            t.start()

        for l in ls:
            l.join()
    print(time.time() - t1)

def main_use_pool():
    """ 使用线程池来优化 """
    t1 = time.time()
    n_list = range(100)
    pool = Pool(10)
    pool.map(run, n_list)
    pool.close()
    pool.join()
    print(time.time() - t1)

def main_use_executor():
    """ 使用 ThreadPoolExecutor 来优化"""
    t1 = time.time()
    n_list = range(100)
    with ThreadPoolExecutor(max_workers=10) as executor:
        executor.map(run, n_list)
    print(time.time() - t1)

if __name__ == '__main__':
    # main() # 同步阻塞
    # main_use_thread() # 20.058868885040283
    # main_use_pool() # 24.130047082901，数据量更大的情况下，比上一种情况更省性能
    main_use_executor() # 20.038520097732544
```
