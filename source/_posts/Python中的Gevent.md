---
title:      Python中的Gevent
---
# Gevent

## 不同的网络模型

- 阻塞式单进程。

一个进程一个循环处理网络请求，当然性能也是最差的
- 阻塞式多进程。

每个请求开一个进程去处理，这样就能同时处理多个请求了， 不过缺点就是当请求数变大时CPU花在进程切换开销巨大，效率低下。
- 非阻塞式事件驱动。

也是多进程，不过使用一个主进程循环来检查是否有网络I/O事件发生，再来决定怎样处理。 省去了上下文切换、进程复制等成本，也不会有死锁、竞争的发生。 不过缺点是没有阻塞式进程直观
- 非阻塞式Coroutine(协程)。

它的本质也是事件驱动，只在单一循环上面检查事件的发生，但是加上了coroutine的概念。 gevent就是这种框架的代表。

## 协程
- 简单来讲，coroutine就是允许你暂时中断之后再继续执行的程序。事实上，python最基础的coroutine就是生成器。

下面代码示例解释了：第一次的next(bar)，执行到了yield，执行两次print后，再执行next(bar)， print(u'foo: 控制权又回到我手上了，叫我大笨蛋')再次停留在yield，再次print(u'main: 老大我又来了')


```python
#!/usr/bin/env python
# -*- coding: utf8 -*-
def foo():
    for i in range(10):
        # 暂时返回一个值，并将控制权交出去
        yield i
        print(u'foo: 控制权又回到我手上了，叫我大笨蛋')


bar = foo()
# 执行coroutine
print(next(bar))
print(u'main: 现在控制权在我手上，啦啦啦')
print('main:hello baby!')
# 回到刚刚foo这个coroutine中断的地方继续执行
print(next(bar))
print(u'main: 老大我又来了')
print(next(bar))
print(next(bar))
```

    0
    main: 现在控制权在我手上，啦啦啦
    main:hello baby!
    foo: 控制权又回到我手上了，叫我大笨蛋
    1
    main: 老大我又来了
    foo: 控制权又回到我手上了，叫我大笨蛋
    2
    foo: 控制权又回到我手上了，叫我大笨蛋
    3

## Gevent

### 原理

- 事实上程序写的跟普通的阻塞式程序一样，但它是异步的
- monkey.patch_all()，也就是猴子补丁。因为python内置的各种函数库和IO库一般都是阻塞式的，比如sleep()就会当前进程，而monkey就是负责将这些阻塞函数全部取代替换成gevent中相应的异步函数。
- gevent打了monkey patch之后会设置python相应的模块设置成非阻塞，然后在内部实现epoll的机制，一旦调用非阻塞的IO(比如recv)都会立刻返回，并且设置一个回调函数，这个回调函数用于切换到当前子coroutine，设置好回掉函数之后就把控制权返回给主coroutine，主coroutine继续调度。一旦网络I/O准备就绪，epoll会触发之前设置的回调函数，从而引发主coroutine切换到子coroutine，做相应的操作。
- joinall()的意思是等待列表中所有coroutine完成后再返回。


```python
"""Spawn multiple workers and wait for them to complete"""
urls = [
    'http://www.gevent.org/', 'http://www.baidu.com', 'http://www.python.org'
]
import gevent
from gevent import monkey
# patches stdlib (including socket and ssl modules) to cooperate with other greenlets
monkey.patch_all()
import urllib2


def print_head(url):
    print 'Starting %s' % url
    data = urllib2.urlopen(url).read()
    print '%s: %s bytes: %r' % (url, len(data), data[:50])


# jobs = list()
# for url in urls:
#     jobs.append(gevent.spawn(print_head, url))
jobs = [gevent.spawn(print_head, url) for url in urls]
gevent.joinall(jobs)
```

    The history saving thread hit an unexpected error (LoopExit('This operation would block forever', <Hub at 0x102fbc050 select pending=0 ref=0>)).History will not be written to the database.
    Starting http://www.gevent.org/
    Starting http://www.baidu.com
    Starting http://www.python.org
    http://www.baidu.com: 112190 bytes: '<!DOCTYPE html>\n<!--STATUS OK-->\n\r\n\r\n\r\n\r\n\r\n\r\n\r\n\r\n\r'
    http://www.python.org: 48853 bytes: '<!doctype html>\n<!--[if lt IE 7]>   <html class="n'
    http://www.gevent.org/: 8734 bytes: '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Trans'

### 上下文切换
gevent.spawn 的重要功能就是封装了greenlet里面的函数。 初始化的greenlet放在了threads这个list里面， 被传递给了 gevent.joinall 这个函数，它会阻塞主程序来执行所有的greenlet。
在异步执行的情况下，所有任务的执行顺序是完全随机的。每一个greenlet的都不会阻塞其他greenlet的执行。

#### 简单示例


```python
import gevent


def foo():
    print('1')
    gevent.sleep(0)
    print('2')


def bar():
    print('3')
    gevent.sleep(0)
    print('4')


gevent.joinall([
    gevent.spawn(foo),
    gevent.spawn(bar),
])
```

    1
    3
    2
    4




    [<Greenlet at 0x10dd045f0>, <Greenlet at 0x10dd04870>]


#### 执行顺序示例代码


```python
import gevent
import random


def task(pid):
    """
    Some non-deterministic task
    """
    gevent.sleep(random.randint(0, 2) * 0.001)
    print('Task', pid, 'done')


def synchronous():
    for i in range(1, 10):
        task(i)


def asynchronous():
    threads = [gevent.spawn(task, i) for i in xrange(10)]
    gevent.joinall(threads)


print('不采用协程的执行顺序，Synchronous:')
synchronous()
print('采用协程的执行顺序，Asynchronous:')
asynchronous()
```

    不采用协程的执行顺序，Synchronous:
    ('Task', 1, 'done')
    ('Task', 2, 'done')
    ('Task', 3, 'done')
    ('Task', 4, 'done')
    ('Task', 5, 'done')
    ('Task', 6, 'done')
    ('Task', 7, 'done')
    ('Task', 8, 'done')
    ('Task', 9, 'done')
    采用协程的执行顺序，Asynchronous:
    ('Task', 1, 'done')
    ('Task', 4, 'done')
    ('Task', 5, 'done')
    ('Task', 7, 'done')
    ('Task', 6, 'done')
    ('Task', 9, 'done')
    ('Task', 3, 'done')
    ('Task', 0, 'done')
    ('Task', 2, 'done')
    ('Task', 8, 'done')

### 生成greenlet
我们通过gevent.spawn来包装了对于greenlet的生成， 另外我们还能可以通过创建Greenlet的子类，并且重写 _run 方法来实现
我的理解是：可以自定义gevent.spawn，协程内部的执行功能


```python
import gevent
from gevent import Greenlet


class MyGreenlet(Greenlet):
    def __init__(self, message, n):
        Greenlet.__init__(self)
        self.message = message
        self.n = n

    def _run(self):
        gevent.sleep(self.n)
        print(self.message)


g = MyGreenlet("Hi there!", 3)
g.start()
g.join()
```

    Hi there!
​    

### Greenlet 的状态
greenlet在执行的时候也会出错。Greenlet有可能会无法抛出异常，停止失败，或者消耗了太多的系统资源。
greenlet的内部状态通常是一个依赖时间的参数。greenlet有一些标记来让你能够监控greenlet的状态
- started – 标志greenlet是否已经启动
- ready – 标志greenlet是否已经被终止
- successful() – 标志greenlet是否已经被终止，并且没有抛出异常
- value – 由greenlet返回的值
- exception – 在greenlet里面没有被捕获的异常


```python
import gevent


def win():
    return 'You win!'


def fail():
    raise Exception('You fail at failing.')


winner = gevent.spawn(win)
loser = gevent.spawn(fail)
print(winner.started)  # True
print(loser.started)  # True
# Exceptions raised in the Greenlet, stay inside the Greenlet.
try:
    gevent.joinall([winner, loser])
except Exception as e:
    print('This will never be reached')
print(winner.value)  # 'You win!'
print(loser.value)  # None
print(winner.ready())  # True
print(loser.ready())  # True
print(winner.successful())  # True
print(loser.successful())  # False
# The exception raised in fail, will not propogate outside the
# greenlet. A stack trace will be printed to stdout but it
# will not unwind the stack of the parent.
print(loser.exception)
```

    True
    True
    You win!
    None
    True
    True
    True
    False
    You fail at failing.

    Traceback (most recent call last):
      File "C:\ProgramData\Anaconda2\lib\site-packages\gevent\greenlet.py", line 536, in run
        result = self._run(*self.args, **self.kwargs)
      File "<ipython-input-8-e403e0ec5a71>", line 9, in fail
        raise Exception('You fail at failing.')
    Exception: You fail at failing.
    Wed Nov 22 10:44:28 2017 <Greenlet at 0x65a6a60L: fail> failed with Exception

​    

### 终止程序
在主程序收到一个SIGQUIT 之后会阻塞程序的执行让Greenlet无法继续执行。 这会导致僵尸进程的产生，需要在操作系统中将这些僵尸进程清除掉。


```python
import gevent
import signal


def run_forever():
    gevent.sleep(1000)


if __name__ == '__main__':
    gevent.signal(signal.SIGQUIT, gevent.shutdown)
    thread = gevent.spawn(run_forever)
    thread.join()
```


    ---------------------------------------------------------------------------
    
    AttributeError                            Traceback (most recent call last)
    
    <ipython-input-10-ebfbe3246457> in <module>()
          8 
          9 if __name__ == '__main__':
    ---> 10     gevent.signal(signal.SIGQUIT, gevent.shutdown)
         11     thread = gevent.spawn(run_forever)
         12     thread.join()

    AttributeError: 'module' object has no attribute 'SIGQUIT'

### 超时
在gevent中支持对于coroutine的超时控制，还能使用with上下文

- 如果协程sleep的时间，小于设置的wait时间，则没有问题


```python
import gevent
from gevent import Timeout
time_to_wait = 6  # seconds


class TooLong(Exception):
    pass


with Timeout(time_to_wait, TooLong):
    gevent.sleep(5)
```

- 如果协程sleep的时间，大于设置的wait时间，则会报异常


```python
import gevent
from gevent import Timeout
time_to_wait = 2  # seconds


class TooLong(Exception):
    pass


with Timeout(time_to_wait, TooLong):
    gevent.sleep(5)
```


    ---------------------------------------------------------------------------
    
    TooLong                                   Traceback (most recent call last)
    
    <ipython-input-13-ac2197c4c150> in <module>()
          9 
         10 with Timeout(time_to_wait, TooLong):
    ---> 11     gevent.sleep(5)

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.py in sleep(seconds, ref)
        167         waiter.get()
        168     else:
    --> 169         hub.wait(loop.timer(seconds, ref=ref))
        170 
        171 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.py in wait(self, watcher)
        649         watcher.start(waiter.switch, unique)
        650         try:
    --> 651             result = waiter.get()
        652             if result is not unique:
        653                 raise InvalidSwitchError('Invalid switch into %s: %r (expected %r)' % (getcurrent(), result, unique))

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.py in get(self)
        897             self.greenlet = getcurrent()
        898             try:
    --> 899                 return self.hub.switch()
        900             finally:
        901                 self.greenlet = None

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.py in switch(self)
        628         if switch_out is not None:
        629             switch_out()
    --> 630         return RawGreenlet.switch(self)
        631 
        632     def switch_out(self):

    TooLong: 

### 猴子补丁
Python的运行时里面允许能够大部分的对象都是可以修改的，包括模块，类和方法。这通常是一个坏主意， 然而在极端的情况下，当有一个库需要加入一些Python基本的功能的时候，monkey patch就能派上用场了。 在上面的例子里，gevent能够改变基础库里的一些使用IO阻塞模型的库比如socket，ssl，threading等等并且把它们改成协程的执行方式。


```python
import socket
print(socket.socket)

print "After monkey patch"
from gevent import monkey
monkey.patch_socket()
print(socket.socket)

import select
print(select.select)

monkey.patch_select()
print "After monkey patch"
print(select.select)
```

    <class 'socket._socketobject'>
    After monkey patch
    <class 'gevent._socket2.socket'>
    <built-in function select>
    After monkey patch
    <function select at 0x000000000632CF28>

### 事件
有时候我们还需要在多个greenlet直接进行通信，比如某些操作的同步。事件(event)是一个在Greenlet之间异步通信的形式。


```python
import gevent
from gevent.event import Event
'''
Illustrates the use of events
'''
evt = Event()


def setter():
    '''After 3 seconds, wake all threads waiting on the value of evt'''
    print('setter: Hey wait for me, I will sleep 3s')
    gevent.sleep(3)
    print("setter: I'm done")
    evt.set()


def waiter():
    '''After 3 seconds the get call will unblock'''
    print("waiter: I'll wait for you")
    evt.wait()  # blocking
    print("waiter: It's my turn")


def main():
    gevent.joinall([
        gevent.spawn(setter),
        gevent.spawn(waiter),
        gevent.spawn(waiter),
        gevent.spawn(waiter),
    ])


if __name__ == '__main__':
    main()
```

    setter: Hey wait for me, I will sleep 3s
    waiter: I'll wait for you
    waiter: I'll wait for you
    waiter: I'll wait for you
    setter: I'm done
    waiter: It's my turn
    waiter: It's my turn
    waiter: It's my turn


```python
import gevent
from gevent.event import AsyncResult
a = AsyncResult()


def setter():
    """
    After 3 seconds set the result of a.
    """
    print 'setter start'
    gevent.sleep(3)
    print 'setter start to set'
    a.set('Hello!')


def waiter():
    """
    After 3 seconds the get call will unblock after the setter
    puts a value into the AsyncResult.
    """
    print 'waiter start'
    print 'waiter start to get'
    print(a.get())


gevent.joinall([
    gevent.spawn(setter),
    gevent.spawn(waiter),
])
```

    getter start to get
    setter start to set
    Hello!




    [<Greenlet at 0x62aa6d0L>, <Greenlet at 0x6367c28L>]


### 队列
队列是一个排序的数据集合，它有常见的put / get操作，但是它是以在Greenlet之间可以安全操作的方式来实现的。
举例来说，如果一个Greenlet从队列中取出一项，此项就不会被同时执行的其它Greenlet再取到了。

#### 阻塞put/get

#### case1
下面的case，在遇到sleep能够执行完毕的原因在于，join方法，会让boss，无论sleep多久，该协程执行完毕之后，再执行下面的joinall,所以可以正常退出。


```python
import gevent
from gevent.queue import Queue
tasks = Queue()


def worker(n):
    while not tasks.empty():
        task = tasks.get()
        print('Worker %s got task %s' % (n, task))
#         gevent.sleep(0)
    print('Quitting time!')


def boss():
    for i in range(1, 25):
        if i == 10:
            gevent.sleep(10)
        tasks.put(i)


gevent.spawn(boss).join()
gevent.joinall([
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'nancy'),
])
```

    Worker steve got task 1
    Worker steve got task 2
    Worker steve got task 3
    Worker steve got task 4
    Worker steve got task 5
    Worker steve got task 6
    Worker steve got task 7
    Worker steve got task 8
    Worker steve got task 9
    Worker steve got task 10
    Worker steve got task 11
    Worker steve got task 12
    Worker steve got task 13
    Worker steve got task 14
    Worker steve got task 15
    Worker steve got task 16
    Worker steve got task 17
    Worker steve got task 18
    Worker steve got task 19
    Worker steve got task 20
    Worker steve got task 21
    Worker steve got task 22
    Worker steve got task 23
    Worker steve got task 24
    Quitting time!
    Quitting time!
    Quitting time!




    [<Greenlet at 0x627c210L>, <Greenlet at 0x627c2a8L>, <Greenlet at 0x627c340L>]


#### case2
boss发送者模型，和消费者worker模型，同时在协程里时：boss会先put 1-9，sleep 10秒时，交出控制权，worker开始运行，等sleep 10 秒过后，继续put，直到全部put完毕，才会get。
协程之间，如果不用sleep交出控制权（不阻塞）的话，会一直执行同一个，直到运行结束。


```python
import gevent
from gevent.queue import Queue
tasks = Queue()


def worker(n):
    while True:
        task = tasks.get()
        print('Worker %s got task %s' % (n, task))
    print('Quitting time!')


def boss():
    for i in range(1, 25):
        if i == 10:
            gevent.sleep(10)
        tasks.put(i)


task_list = [
    gevent.spawn(boss),
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'nancy'),
]
gevent.joinall(task_list)
```

    Worker steve got task 1
    Worker steve got task 2
    Worker steve got task 3
    Worker steve got task 4
    Worker steve got task 5
    Worker steve got task 6
    Worker steve got task 7
    Worker steve got task 8
    Worker steve got task 9
    Worker steve got task 10
    Worker steve got task 11
    Worker steve got task 12
    Worker steve got task 13
    Worker steve got task 14
    Worker steve got task 15
    Worker steve got task 16
    Worker steve got task 17
    Worker steve got task 18
    Worker steve got task 19
    Worker steve got task 20
    Worker steve got task 21
    Worker steve got task 22
    Worker steve got task 23
    Worker steve got task 24


    ---------------------------------------------------------------------------
    
    LoopExit                                  Traceback (most recent call last)
    
    <ipython-input-6-8e3bd6d0d8c4> in <module>()
         29 
         30 # boss()
    ---> 31 gevent.joinall(task_list)

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\greenlet.pyc in joinall(greenlets, timeout, raise_error, count)
        647     """
        648     if not raise_error:
    --> 649         return wait(greenlets, timeout=timeout, count=count)
        650 
        651     done = []

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in wait(objects, timeout, count)
       1036     if objects is None:
       1037         return get_hub().join(timeout=timeout)
    -> 1038     return list(iwait(objects, timeout, count))
       1039 
       1040 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in iwait(objects, timeout, count)
        983 
        984         for _ in xrange(count):
    --> 985             item = waiter.get()
        986             waiter.clear()
        987             if item is _NONE:

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        937     def get(self):
        938         if not self._values:
    --> 939             Waiter.get(self)
        940             Waiter.clear(self)
        941 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        897             self.greenlet = getcurrent()
        898             try:
    --> 899                 return self.hub.switch()
        900             finally:
        901                 self.greenlet = None

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in switch(self)
        628         if switch_out is not None:
        629             switch_out()
    --> 630         return RawGreenlet.switch(self)
        631 
        632     def switch_out(self):

    LoopExit: ('This operation would block forever', <Hub at 0x627c048 select default pending=0 ref=0>)

下面的代码，等同于上面的效果


```python
import gevent
from gevent.queue import Queue
tasks = Queue()


def worker(n):
    while True:
        task = tasks.get()
        print('Worker %s got task %s' % (n, task))
    print('Quitting time!')


def boss():
    for i in range(1, 25):
        if i == 10:
            gevent.sleep(10)
        tasks.put(i)


task_list = [
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'nancy'),
]

boss()
gevent.joinall(task_list)
```

    Worker steve got task 1
    Worker steve got task 2
    Worker steve got task 3
    Worker steve got task 4
    Worker steve got task 5
    Worker steve got task 6
    Worker steve got task 7
    Worker steve got task 8
    Worker steve got task 9
    Worker steve got task 10
    Worker steve got task 11
    Worker steve got task 12
    Worker steve got task 13
    Worker steve got task 14
    Worker steve got task 15
    Worker steve got task 16
    Worker steve got task 17
    Worker steve got task 18
    Worker steve got task 19
    Worker steve got task 20
    Worker steve got task 21
    Worker steve got task 22
    Worker steve got task 23
    Worker steve got task 24


    ---------------------------------------------------------------------------
    
    LoopExit                                  Traceback (most recent call last)
    
    <ipython-input-7-00652269a509> in <module>()
         26 
         27 boss()
    ---> 28 gevent.joinall(task_list)

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\greenlet.pyc in joinall(greenlets, timeout, raise_error, count)
        647     """
        648     if not raise_error:
    --> 649         return wait(greenlets, timeout=timeout, count=count)
        650 
        651     done = []

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in wait(objects, timeout, count)
       1036     if objects is None:
       1037         return get_hub().join(timeout=timeout)
    -> 1038     return list(iwait(objects, timeout, count))
       1039 
       1040 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in iwait(objects, timeout, count)
        983 
        984         for _ in xrange(count):
    --> 985             item = waiter.get()
        986             waiter.clear()
        987             if item is _NONE:

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        937     def get(self):
        938         if not self._values:
    --> 939             Waiter.get(self)
        940             Waiter.clear(self)
        941 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        897             self.greenlet = getcurrent()
        898             try:
    --> 899                 return self.hub.switch()
        900             finally:
        901                 self.greenlet = None

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in switch(self)
        628         if switch_out is not None:
        629             switch_out()
    --> 630         return RawGreenlet.switch(self)
        631 
        632     def switch_out(self):

    LoopExit: ('This operation would block forever', <Hub at 0x627c048 select default pending=0 ref=0>)

#### case3
下面的case不同于case2，boss相当于并没有采用协程的概念，即使sleep了，也全部放入队列里之后，才会执行下面的消费者协程


```python
import gevent
from gevent.queue import Queue
tasks = Queue()


def worker(n):
    while True:
        task = tasks.get()
        print('Worker %s got task %s' % (n, task))
    print('Quitting time!')


def boss():
    for i in range(1, 25):
        if i == 10:
            gevent.sleep(10)
        tasks.put(i)


boss()

task_list = [
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'nancy'),
]
gevent.joinall(task_list)
```

    Worker steve got task 1
    Worker steve got task 2
    Worker steve got task 3
    Worker steve got task 4
    Worker steve got task 5
    Worker steve got task 6
    Worker steve got task 7
    Worker steve got task 8
    Worker steve got task 9
    Worker steve got task 10
    Worker steve got task 11
    Worker steve got task 12
    Worker steve got task 13
    Worker steve got task 14
    Worker steve got task 15
    Worker steve got task 16
    Worker steve got task 17
    Worker steve got task 18
    Worker steve got task 19
    Worker steve got task 20
    Worker steve got task 21
    Worker steve got task 22
    Worker steve got task 23
    Worker steve got task 24


    ---------------------------------------------------------------------------
    
    LoopExit                                  Traceback (most recent call last)
    
    <ipython-input-8-e77d06944ce9> in <module>()
         26     gevent.spawn(worker, 'nancy'),
         27 ]
    ---> 28 gevent.joinall(task_list)

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\greenlet.pyc in joinall(greenlets, timeout, raise_error, count)
        647     """
        648     if not raise_error:
    --> 649         return wait(greenlets, timeout=timeout, count=count)
        650 
        651     done = []

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in wait(objects, timeout, count)
       1036     if objects is None:
       1037         return get_hub().join(timeout=timeout)
    -> 1038     return list(iwait(objects, timeout, count))
       1039 
       1040 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in iwait(objects, timeout, count)
        983 
        984         for _ in xrange(count):
    --> 985             item = waiter.get()
        986             waiter.clear()
        987             if item is _NONE:

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        937     def get(self):
        938         if not self._values:
    --> 939             Waiter.get(self)
        940             Waiter.clear(self)
        941 

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in get(self)
        897             self.greenlet = getcurrent()
        898             try:
    --> 899                 return self.hub.switch()
        900             finally:
        901                 self.greenlet = None

    C:\ProgramData\Anaconda2\lib\site-packages\gevent\hub.pyc in switch(self)
        628         if switch_out is not None:
        629             switch_out()
    --> 630         return RawGreenlet.switch(self)
        631 
        632     def switch_out(self):

    LoopExit: ('This operation would block forever', <Hub at 0x627c048 select default pending=0 ref=0>)

#### case4
正确的退出方式应该是这样的！！！


```python
import gevent
from gevent.queue import Queue
tasks = Queue(maxsize=5)
Thread_SIZE = 3
            
def worker():
    while True:
        try:
            task = tasks.get()
            if task == "stop":
                break
            print('Worker got task %s' % (task))
        except Exception as e:
            print u"error : " + str(e)
    print('Quitting time!')


def boss():
    for i in range(1, 10):
        tasks.put(i)
    for i in range(Thread_SIZE):
        tasks.put("stop")
   
task_list = list()
task_list.append(gevent.spawn(boss))
for i in range(Thread_SIZE):
    task_list.append(gevent.spawn(worker))
    
gevent.joinall(task_list)
```

    Worker got task 1
    Worker got task 2
    Worker got task 3
    Worker got task 4
    Worker got task 5
    Worker got task 6
    Worker got task 7
    Worker got task 8
    Worker got task 9
    Quitting time!
    Quitting time!
    Quitting time!




    [<Greenlet at 0x652a210L>,
     <Greenlet at 0x652a178L>,
     <Greenlet at 0x652a2a8L>,
     <Greenlet at 0x652a340L>]


#### 阻塞 put_nowait和get_nowait

在操作不能完成时抛出gevent.queue.Empty或gevent.queue.Full异常

#### queue 最大长度

- 不设置Queue(maxsize=3)，在发送协程没有阻塞的时候，持续发送，直到完毕。


```python
import gevent
from gevent.queue import Queue, Empty
tasks = Queue()

def worker(n):
    try:
        while True:
            task = tasks.get(timeout=1) # decrements queue size by 1
            print('Worker %s got task %s' % (n, task))
            gevent.sleep(0)
    except Empty:
        print('Quitting time!')
        
def boss():
    """
    Boss will wait to hand out work until a individual worker is
    free since the maxsize of the task queue is 3.
    """
    for i in xrange(1,10):
        tasks.put(i)
        print u"tasks size : " + str(tasks.qsize())
    print('Assigned all work in iteration 1')
    for i in xrange(10,20):
        tasks.put(i)
        print u"tasks size : " + str(tasks.qsize())
    print('Assigned all work in iteration 2')
    
gevent.joinall([
    gevent.spawn(boss),
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'bob'),
])
```

    tasks size : 1
    tasks size : 2
    tasks size : 3
    tasks size : 4
    tasks size : 5
    tasks size : 6
    tasks size : 7
    tasks size : 8
    tasks size : 9
    Assigned all work in iteration 1
    tasks size : 10
    tasks size : 11
    tasks size : 12
    tasks size : 13
    tasks size : 14
    tasks size : 15
    tasks size : 16
    tasks size : 17
    tasks size : 18
    tasks size : 19
    Assigned all work in iteration 2
    Worker steve got task 1
    Worker john got task 2
    Worker bob got task 3
    Worker steve got task 4
    Worker john got task 5
    Worker bob got task 6
    Worker steve got task 7
    Worker john got task 8
    Worker bob got task 9
    Worker steve got task 10
    Worker john got task 11
    Worker bob got task 12
    Worker steve got task 13
    Worker john got task 14
    Worker bob got task 15
    Worker steve got task 16
    Worker john got task 17
    Worker bob got task 18
    Worker steve got task 19
    Quitting time!
    Quitting time!
    Quitting time!




    [<Greenlet at 0x1055f1690>,
     <Greenlet at 0x1055f1870>,
     <Greenlet at 0x1055f1f50>,
     <Greenlet at 0x1055f12d0>]


- 设置Queue(maxsize=3)，当发送协程queue为3时，会暂停，交出控制权，让消费者去消耗，小于3时，再put.


```python
import gevent
from gevent.queue import Queue, Empty
tasks = Queue(maxsize=3)

def worker(n):
    try:
        while True:
            task = tasks.get(timeout=10) # decrements queue size by 1
            print('Worker %s got task %s' % (n, task))
            gevent.sleep(0)
    except Empty:
        print('Quitting time!')
        
def boss():
    """
    Boss will wait to hand out work until a individual worker is
    free since the maxsize of the task queue is 3.
    """
    for i in xrange(1,10):
        tasks.put(i)
        print u"tasks size : " + str(tasks.qsize())
    print('Assigned all work in iteration 1')
    for i in xrange(10,20):
        tasks.put(i)
        print u"tasks size : " + str(tasks.qsize())
    print('Assigned all work in iteration 2')
    
gevent.joinall([
    gevent.spawn(boss),
    gevent.spawn(worker, 'steve'),
    gevent.spawn(worker, 'john'),
    gevent.spawn(worker, 'bob'),
])
```

    tasks size : 1
    tasks size : 2
    tasks size : 3
    Worker steve got task 1
    Worker john got task 2
    Worker bob got task 3
    tasks size : 1
    tasks size : 2
    tasks size : 3
    Worker steve got task 4
    Worker john got task 5
    Worker bob got task 6
    tasks size : 1
    tasks size : 2
    tasks size : 3
    Assigned all work in iteration 1
    Worker steve got task 7
    Worker john got task 8
    Worker bob got task 9
    tasks size : 1
    tasks size : 2
    tasks size : 3
    Worker steve got task 10
    Worker john got task 11
    Worker bob got task 12
    tasks size : 1
    tasks size : 2
    tasks size : 3
    Worker steve got task 13
    Worker john got task 14
    Worker bob got task 15
    tasks size : 1
    tasks size : 2
    tasks size : 3
    Worker steve got task 16
    Worker john got task 17
    Worker bob got task 18
    tasks size : 1
    Assigned all work in iteration 2
    Worker steve got task 19
    Quitting time!
    Quitting time!
    Quitting time!




    [<Greenlet at 0x1055f1410>,
     <Greenlet at 0x1055e3c30>,
     <Greenlet at 0x1055e3e10>,
     <Greenlet at 0x1055e3550>]


#### queue timeout
能够保证在get不到的10秒时间里，正常退出

### 组


```python
import gevent
from gevent.pool import Group

def talk(msg):
    for i in xrange(3):
        print(msg)
        
g1 = gevent.spawn(talk, 'bar')
g2 = gevent.spawn(talk, 'foo')
g3 = gevent.spawn(talk, 'fizz')
group = Group()
group.add(g1)
group.add(g2)
print u"g1, g2 开始join"
group.join()
group.add(g3)
print u"g3 开始join"
group.join()
```

    g1, g2 开始join
    bar
    bar
    bar
    foo
    foo
    foo
    fizz
    fizz
    fizz
    g3 开始join




    True


### 池


```python
from gevent.pool import Pool
class SocketPool(object):
    
    def __init__(self):
        self.pool = Pool(1000)
        self.pool.start()
        
    def listen(self, socket):
        while True:
            socket.recv()
            
    def add_handler(self, socket):
        if self.pool.full():
            raise Exception("At maximum pool size")
        else:
            self.pool.spawn(self.listen, socket)
            
    def shutdown(self):
        self.pool.kill()
```


```python
import gevent
from gevent.pool import Pool
pool = Pool(2)


def hello_from(n):
    print('Size of pool %s' % len(pool))
    
pool.map(hello_from, xrange(3))
```

    Size of pool 2
    Size of pool 2
    Size of pool 1




    [None, None, None]



```python
import time
import gevent
from gevent.threadpool import ThreadPool
 
def my_func(text, num):
    print text, num
 
pool = ThreadPool(2)
start = time.time()
for i in xrange(10):
    pool.spawn(my_func, "Hello", i)
pool.join()
delay = time.time() - start
print('Take %.3f seconds' % delay)
```

    HelloHello  01
    
    Hello  Hello2 
    3Hello
     Hello4 
    5Hello
    Take 0.013 seconds Hello
    6 
    7Hello
      8Hello
     9


```python
import time
import gevent.pool

pool=gevent.pool.Pool(2)

def func(n):
    print('Size of pool %s' % len(pool))
    print n

for i in range(10):
    pool.add(gevent.spawn(func,i))

pool.join()
```

    Size of pool 2
    0
    Size of pool 2
    1
    Size of pool 2
    2
    Size of pool 1
    3
    Size of pool 1
    4
    Size of pool 0
    5
    Size of pool 1
    6
    Size of pool 1
    7
    Size of pool 0
    8
    Size of pool 1
    9




    True



```python

```
