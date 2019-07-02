---
title:      Python中的多进程和多进程
---

# 多线程和多进程

## 多线程（Threading）
多线程的理念是分批的想法。<br>
如果我有一大批数据，使用一个cpu，可能需要10秒，但是如果我把数据分成两批，同时使用2个CPU进行处理，时间就可以减少到5秒。<br>

### 添加线程


```python
import threading


def thread_job():
    print('This is added Thread, number is %s' % threading.current_thread())


def main():
    print(threading.active_count())
    print(threading.enumerate())
    print(threading.current_thread())

    add_thread = threading.Thread(target=thread_job)
    add_thread.start()

    print(threading.active_count())
    print(threading.enumerate())
    print(threading.current_thread())


if __name__ == '__main__':
    main()
```

    5This is added Thread, number is <Thread(Thread-7, started 123145512390656)>
    
    [<_MainThread(MainThread, started 4485215680)>, <Thread(Thread-2, started daemon 123145490296832)>, <Heartbeat(Thread-3, started daemon 123145495552000)>, <HistorySavingThread(IPythonHistorySavingThread, started 123145501880320)>, <ParentPollerUnix(Thread-1, started daemon 123145507135488)>]
    <_MainThread(MainThread, started 4485215680)>
    5
    [<_MainThread(MainThread, started 4485215680)>, <Thread(Thread-2, started daemon 123145490296832)>, <Heartbeat(Thread-3, started daemon 123145495552000)>, <HistorySavingThread(IPythonHistorySavingThread, started 123145501880320)>, <ParentPollerUnix(Thread-1, started daemon 123145507135488)>]
    <_MainThread(MainThread, started 4485215680)>


### join


```python
import threading
import time


def thread_job():
    print('T1 Start \n')
    for i in range(10):
        time.sleep(0.1)
    print('T1 Finish \n')


def main():
    add_thread = threading.Thread(target=thread_job, name='T1')
    add_thread.start()
    
    print('all done \n')


if __name__ == '__main__':
    main()
```

    all done 
    T1 Start 
    T1 Finish 




```python
import threading
import time


def thread_job():
    print('T1 Start \n')
    for i in range(10):
        time.sleep(0.1)
    print('T1 Finish \n')


def main():
    add_thread = threading.Thread(target=thread_job, name='T1')
    add_thread.start()

    add_thread.join()

    print('all done \n')


if __name__ == '__main__':
    main()
```

    T1 Start 
    
    T1 Finish 
    all done 


​    



```python
import threading
import time


def thread_job():
    print('T1 Start \n')
    for i in range(10):
        time.sleep(0.1)
    print('T1 Finish \n')


def thread_job2():
    print('T2 Start \n')
    print('T2 Finish \n')


def main():
    add_thread = threading.Thread(target=thread_job, name='T1')
    add_thread2 = threading.Thread(target=thread_job2, name='T2')
    add_thread.start()
    add_thread.join()

    add_thread2.start()
    add_thread2.join()

    print('all done \n')


if __name__ == '__main__':
    main()
```

    T1 Start 
    
    T1 Finish 
    T2 Start 
    all done 
    
    T2 Finish 


  

```python
import threading
import time


def thread_job():
    print('T1 Start \n')
    for i in range(10):
        time.sleep(0.1)
    print('T1 Finish \n')


def thread_job2():
    print('T2 Start \n')
    print('T2 Finish \n')


def main():
    add_thread = threading.Thread(target=thread_job, name='T1')
    add_thread2 = threading.Thread(target=thread_job2, name='T2')
    add_thread.start()
    add_thread2.start()

    add_thread.join()
    add_thread2.join()

    print('all done \n')


if __name__ == '__main__':
    main()
```

    T1 Start 
    T2 Start 
    T2 Finish 
    
    T1 Finish 
    all done  


### Queue


```python
import time
import threading
from queue import Queue


def job(l, q):
    for i in range(len(l)):
        l[i] = l[i]**2
    q.put(l)


def multithreading(data):
    q = Queue()
    threads = []
    for i in range(4):
        t = threading.Thread(target=job, args=(data[i], q))
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join()
    result = list()

    for _ in range(4):
        result.append(q.get())
    print(result)


if __name__ == '__main__':
    data = [[1, 1, 1], [2, 2, 2], [3, 3, 3], [4, 4, 4]]
    multithreading(data)
```

    [[1, 1, 1], [4, 4, 4], [9, 9, 9], [16, 16, 16]]


### GIL不一定有效率

python中的多线程，其实并不并行的，在多线程运行的时候，GIL会把其他的线程锁住，只让一个线程在同一时间运行一个东西。<br>
本质上，python的多线程其实是在多个线程之间切换的。而不是并行的。


```python
import threading
from queue import Queue
import copy
import time


def job(l, q):
    res = sum(l)
    q.put(res)


def multithreading(l):
    q = Queue()
    threads = []
    for i in range(4):
        t = threading.Thread(
            target=job, args=(copy.copy(l), q), name='T%i' % i)
        t.start()
        threads.append(t)
    [t.join() for t in threads]
    total = 0
    for _ in range(4):
        total += q.get()
    print(total)


def normal(l):
    total = sum(l)
    print(total)


if __name__ == '__main__':
    l = list(range(1000000))
    s_t = time.time()
    normal(l * 4)
    print('normal: ', time.time() - s_t)
    s_t = time.time()
    multithreading(l)
    print('multithreading: ', time.time() - s_t)
```

    1999998000000
    normal:  0.09590601921081543
    1999998000000
    multithreading:  0.07823991775512695


### 线程锁 Lock
全局变量线程不安全 ！！！


```python
import threading
import time


class A(object):
    def __init__(self):
        self.value = 1


def thread1(a):
    a.value = a.value + 1
    print('T1, stage 1, a : {0} \n'.format(a.value))
    time.sleep(0.1)
    print('T1, stage 2, a : {0} \n'.format(a.value))


def thread2(a):
    a.value = a.value + 1
    print('T2, stage 1, a : {0} \n'.format(a.value))
    time.sleep(0.1)
    print('T2, stage 2, a : {0} \n'.format(a.value))


if __name__ == '__main__':
    a = A()
    Thread1 = threading.Thread(target=thread1, name='T1', args=(a, ))
    Thread2 = threading.Thread(target=thread2, name='T2', args=(a, ))
    Thread1.start()
    Thread2.start()
    Thread1.join()
    Thread2.join()
```

    T1, stage 1, a : 2 
    
    T2, stage 1, a : 3 
    
    T1, stage 2, a : 3 
    
    T2, stage 2, a : 3 




```python
import threading
import time


def job1():
    global A
    for i in range(10):
        A += 1
        print('job1 {0} \n'.format(A))
        time.sleep(0.1)


def job2():
    global A
    for i in range(10):
        A += 10
        print('job2 {0} \n'.format(A))
        time.sleep(0.1)


if __name__ == '__main__':
    lock = threading.Lock()
    A = 0
    t1 = threading.Thread(target=job1)
    t2 = threading.Thread(target=job2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

    job1 1 
    job2 11 
    job1 12 
    job2 22 
    job1 23 
    job2 33 
    job1 34 
    job2 44 
    job1 45 
    job2 55 
    job1 56 
    job2 66 
    job1 67 
    job2 77 
    job2 87 
    job1 88 
    job2 98 
    job1 99 
    job2 109 
    job1 110     



```python
import threading
import time


def job1():
    global A, lock
    lock.acquire()
    for i in range(10):
        A += 1
        print('job1 {0} \n'.format(A))
        time.sleep(0.1)
    lock.release()


def job2():
    global A, lock
    lock.acquire()
    for i in range(10):
        A += 10
        print('job2 {0} \n'.format(A))
        time.sleep(0.1)
    lock.release()


if __name__ == '__main__':
    lock = threading.Lock()
    A = 0
    t1 = threading.Thread(target=job1)
    t2 = threading.Thread(target=job2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

    job1 1 
    
    job1 2 
    
    job1 3 
    
    job1 4 
    
    job1 5 
    
    job1 6 
    
    job1 7 
    
    job1 8 
    
    job1 9 
    
    job1 10 
    
    job2 20 
    
    job2 30 
    
    job2 40 
    
    job2 50 
    
    job2 60 
    
    job2 70 
    
    job2 80 
    
    job2 90 
    
    job2 100 
    
    job2 110 



## 多进程

### 创建进程


```python
import multiprocessing as mp
import threading as td


def job(a, b):
    print(a, b)


t1 = td.Thread(target=job, args=(1, 2))
p1 = mp.Process(target=job, args=(1, 2))

t1.start()
p1.start()
t1.join()
p1.join()
```

    1 2
    1 2



```python
import multiprocessing as mp


def job(a, b):
    print(a, b)


if __name__ == '__main__':
    p1 = mp.Process(target=job, args=(1, 2))
    p1.start()
    p1.join()
```

    1 2


### Queue


```python
import multiprocessing as mp


def job(q):
    res = 0
    for i in range(1000):
        res += i + i**2 + i**3
    q.put(res)  #queue


if __name__ == '__main__':
    q = mp.Queue()
    p1 = mp.Process(target=job, args=(q, ))
    p2 = mp.Process(target=job, args=(q, ))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    res1 = q.get()
    res2 = q.get()
    print(res1 + res2)
```

    499667166000


### 效率对比


```python
import multiprocessing as mp
import threading as td
import time


def normal():
    res = 0
    for _ in range(2):
        for i in range(1000000):
            res += i + i**2 + i**3
    print('normal:', res)


def job(q):
    res = 0
    for i in range(1000000):
        res += i + i**2 + i**3
    q.put(res)  # queue


def multithread():
    q = mp.Queue()  # thread可放入process同样的queue中
    t1 = td.Thread(target=job, args=(q, ))
    t2 = td.Thread(target=job, args=(q, ))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    res1 = q.get()
    res2 = q.get()
    print('multithread:', res1 + res2)


def multicore():
    q = mp.Queue()
    p1 = mp.Process(target=job, args=(q, ))
    p2 = mp.Process(target=job, args=(q, ))
    p1.start() 
    p2.start()
    p1.join()
    p2.join()
    res1 = q.get()
    res2 = q.get()
    print('multicore:', res1 + res2)


if __name__ == '__main__':
    st = time.time()
    normal()
    st1 = time.time()
    print('normal time:', st1 - st)
    multithread()
    st2 = time.time()
    print('multithread time:', st2 - st1)
    multicore()
    print('multicore time:', time.time() - st2)
```

    normal: 499999666667166666000000
    normal time: 1.3003261089324951
    multithread: 499999666667166666000000
    multithread time: 1.2527809143066406
    multicore: 499999666667166666000000
    multicore time: 0.645003080368042


### 进程池


```python
import multiprocessing as mp


def job(x):
    return x * x


def multicore():
    pool = mp.Pool(processes=3)
    result = pool.map(job, range(10))
    print(result)

    res = pool.apply_async(job, (2, ))
    print(res.get())

    multi_res = [pool.apply_async(job, (i, )) for i in range(10)]
    print([res.get() for res in multi_res])


if __name__ == '__main__':
    multicore()
```

    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
    4
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]


### 共享内存
多进程之间是独立的，在不做任何处理的情况下，是无法共同访问和修改同一个变量的。


```python
import multiprocessing as mp

# 设置共享内存，多核和多进程可以同时访问
value = mp.Value('d',0)
array = mp.Array('i',[1,2,3])
```

### 进程锁


```python
import multiprocessing as mp
import time


def job1(A):
    for i in range(10):
        A.value += 1
        print('job1: {0} \n'.format(A.value))
        time.sleep(0.1)


def job2(A):
    for i in range(10):
        A.value += 1
        print('job2: {0} \n'.format(A.value))
        time.sleep(0.1)


if __name__ == '__main__':
    A = mp.Value('i', 0)
    print(A)
    p1 = mp.Process(target=job1, args=(A, ))
    p2 = mp.Process(target=job2, args=(A, ))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

    <Synchronized wrapper for c_int(0)>
    job1: 1 
    
    job2: 2 
    
    job1: 3 
    
    job2: 4 
    
    job1: 5 
    
    job2: 6 
    
    job1: 7 
    
    job2: 8 
    
    job1: 9 
    
    job2: 10 
    
    job1: 11 
    
    job2: 12 
    
    job1: 13 
    
    job2: 13 
    
    job1: 14 
    
    job2: 15 
    
    job1: 16 
    
    job2: 17 
    
    job1: 18 
    
    job2: 18 


​    
```python
import multiprocessing as mp
import time


def job1(A, lock):
    lock.acquire()
    for i in range(10):
        A.value += 1
        print('job1: {0} \n'.format(A.value))
        time.sleep(0.1)
    lock.release()


def job2(A, lock):
    lock.acquire()
    for i in range(10):
        A.value += 1
        print('job2: {0} \n'.format(A.value))
        time.sleep(0.1)
    lock.release()


if __name__ == '__main__':
    lock = mp.Lock()
    A = mp.Value('i', 0)
    print(A)
    p1 = mp.Process(target=job1, args=(A, lock))
    p2 = mp.Process(target=job2, args=(A, lock))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

    <Synchronized wrapper for c_int(0)>
    job1: 1 
    
    job1: 2 
    
    job1: 3 
    
    job1: 4 
    
    job1: 5 
    
    job1: 6 
    
    job1: 7 
    
    job1: 8 
    
    job1: 9 
    
    job1: 10 
    
    job2: 11 
    
    job2: 12 
    
    job2: 13 
    
    job2: 14 
    
    job2: 15 
    
    job2: 16 
    
    job2: 17 
    
    job2: 18 
    
    job2: 19 
    
    job2: 20 


