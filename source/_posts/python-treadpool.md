---
title: python多线程及threadpoolExecutor线程池的使用
date: 2018-08-27 18:12:33
tags:
---

从python3.2 python的标准库内置了concurrent.futures模块,其内置了ThreadPoolExecutor和ProcessPoolExecutor两个类，实现了对threading(多线程)和multiprocessing(多进程)的进一步抽象.


<!-- more -->

### 线程的基本操作
```python
'''
利用网络爬虫的例子来使用线程池, 我们定义函数`get_html()`来模拟网络爬虫, 
且通过`times`参数模拟所消耗的时间来更清晰的展现线程的运行状态.
'''
def get_html(times):
    time.sleep(times)
    print("get page {}s finished".format(times))
    return times
    
# 创建线程池
# 参数: max_workers 最多可运行的线程数量
executor = ThreadPoolExecutor(max_workers=2)

# `submit` - 提交任务
task1 = executor.submit(get_html, (4))
task2 = executor.submit(get_html, (5))
task3 = executor.submit(get_html, (6))
'''
submit方法是不阻塞的, 调用之后立即返回, 
所以这个地方我们一共提交了三个线程(task1, task2, task3), 
但是最大的线程数为2, 所以此时会有一个线程在等待中.
'''


# `done` - 判定任务是否完成
print(task1.done())

# `cancel` - 取消某个任务, 只能取消正在等待的线程, 也就是task3, 否则会取消失败.
print(task2.cancel())
print(task3.cancel())
time.sleep(4)
print(task1.done())

# `result` - 获取task的执行结果
print(task1.result())
```


-----------
### as_completed 
as_completed()方法是一个生成器，在没有任务完成的时候，会阻塞，在有某个任务完成的时候，会yield这个任务，就能执行for循环下面的语句，然后继续阻塞住，循环到所有的任务结束。从结果也可以看出，先完成的任务会先通知主线程。

```python
urls = [8, 6, 4, 10]
alltasks = [executor.submit(get_html, (url)) for url in urls]

for future in as_completed(alltasks):
    data = future.result()
    print("in main: get page {}s success".format(data))
    
# *输出结果*
# get page 4s finished
# in main: get page 4s success
# get page 6s finished
# in main: get page 6s success
# get page 8s finished
# in main: get page 8s success
# get page 10s finished
# in main: get page 10s success

```
-----------

### map 
使用map方法，无需提前使用submit方法, map方法与python标准库中的map含义相同, 都是将序列中的每个元素都执行同一个函数.
```python
# map 同样监控线程的执行结果,但是map会根据列表的顺序去输出线程的结果
urls = [8, 6, 4, 10]

alltasks = [executor.submit(get_html, url) for url in urls]

for data in executor.map(get_html, urls):
    print("in main: get page {}s success".format(data))
    
# 输出结果
# get page 4s finished
# get page 6s finished
# get page 8s finished
# in main: get page 8s success
# in main: get page 6s success
# in main: get page 4s success
# get page 10s finished
# in main: get page 10s success

```

-----------
### wait
wait方法可以让主线程阻塞，直到满足设定的要求。
```python
# 第四步: wait 根据状态去等待进程, wait会阻塞进程直到线程达到指定的状态

# wait
# 参数: fs 任务
#      timeout 等待超时时间
#      return_when (ALL_COMPLETED, FIRST_COMPLETED, FIRST_EXCEPTION) 等待条件  默认ALL_COMPLETED
       
urls = [8, 6, 4, 10]
alltasks = [executor.submit(get_html, url) for url in urls]
wait(alltasks, return_when=ALL_COMPLETED)
print("main")

# 输出结果
# get page 4s finished
# get page 6s finished
# get page 8s finished
# get page 10s finished
# main
```

此次就只讲线程池的使用, 多线程的其他知识下次更新
