---
layout: post
title: python的代码测速方法推荐
description: python的代码测速方法推荐
category: 技术
tags: [python]
---

*最后修改时间: 2019-10-24*

## 1. 缘起
对于python中的脚本进行测速，自己之前一致用decorator里面写time.time()-start_time的方式来测速，偶然看到了一些不同高级的方法，但也没有特别系统的文章，所以记录一下。

## 2. 不同的测速方式
```py
def calculate():
    import math
    a = [math.sqrt(i) for i in range(100000)]
```
#### 2.1 ipython的magic command
1. line mode: ```%timeit calculate()```
2. cell mode: 
```
%%timeit 
calculate()
```

ipython的魔术方法可以自动决定循环次数然后输出最好的三个结果(最好的结果具有代表性)

#### 2.2 timeit的标准用法
```py
from timeit import timeit
print('time Usage {} s'.format(timeit(calculate, number=1)))
```

#### 2.3 自己造decorator
```py
import functools
import time
def timeit(func):
    @functools.wraps(func)
    def newfunc(*args, **kwargs):
        startTime = time.time()
        func(*args, **kwargs)
        elapsedTime = time.time() - startTime
        print('function [{}] finished in {} ms'.format(
            func.__name__, int(elapsedTime * 1000)))
    return newfunc
@timeit
def calculate():
    import math
    a = [math.sqrt(i) for i in range(100000)]
calculate()
```

#### 2.4 自己造context manager(不需要放入function就能测速)
```py
from contextlib import contextmanager
@contextmanager
def timeit_context(name):
    startTime = time.time()
    yield
    elapsedTime = time.time() - startTime
    print('[{}] finished in {} ms'.format(name, int(elapsedTime * 1000)))
with timeit_context('My code'):
    import math
    a = [math.sqrt(i) for i in range(10000)]
    b = [math.sqrt(i) for i in range(100000)]
```

#### 2.5 line_profile的非magic方法
需要pip install line_profile, 这是个好东西，可以计算每一行的运行时间，但是官方推荐的方法是通过kernprof命令使用，不太舒服，找到一种pythonic的方法，作者adisonhuang甚至实现了可以传入子方法

```py
from functools import  wraps
try:
    from line_profiler import LineProfiler
    def func_line_time(follow=[]):
        """
        每行代码执行时间详细报告
        :param follow: 内部调用方法
        :return:
        """
        def decorate(func):
            @wraps(func)
            def profiled_func(*args, **kwargs):
                try:
                    profiler = LineProfiler()
                    profiler.add_function(func)
                    for f in follow:
                        profiler.add_function(f)
                    profiler.enable_by_count()
                    return func(*args, **kwargs)
                finally:
                    profiler.print_stats()

            return profiled_func

        return decorate

except ImportError:
    def func_line_time(follow=[]):
        "Helpful if you accidentally leave in production!"
        def decorate(func):
            @wraps(func)
            def nothing(*args, **kwargs):
                return func(*args, **kwargs)

            return nothing

        return decorate

def sub_calculate():
    import math
    b = [math.sqrt(i) for i in range(10000)]

@func_line_time([sub_calculate])
def calculate():
    import math
    a = [math.sqrt(i) for i in range(100000)]
    sub_calculate()

calculate()
```

#### 2.6 命令行/usr/bin/time -p python main.py

#### 2.7 使用memory_profiler 测内存使用量
需要pip install memory_profiler
```
from memory_profiler import profile

@profile
def calculate():
    import math
    a = [math.sqrt(i) for i in range(100000)]
    b = [math.sqrt(i) for i in range(100000)]
calculate()
```

## 3. 总结

+ 在ipython下%timeit是最方便的方式
+ 一般写脚本的时候decorator和context manager是最方便的
+ 对全脚本测试用/usr/bin/time
+ 要对每一行进行测速/测内容实用line_profiler/memory_profiler

## 4. 参考
+ [stackoverflow](https://stackoverflow.com/a/20924212)
+ [python_timeit](https://docs.python.org/zh-cn/3/library/timeit.html)
+ [ipython_magic](https://ipython.org/ipython-doc/dev/interactive/magics.html)
+ [python基础性能测试](https://nullcc.github.io/2018/02/22/Python性能测试基础/)
+ [stackoverflow](https://stackoverflow.com/questions/23885147/how-do-i-use-line-profiler-from-robert-kern)
+ [python-line-profiler-without-magic](https://lothiraldan.github.io/2018-02-18-python-line-profiler-without-magic/)
+ [python性能优化之函数执行时间分析](https://www.jianshu.com/p/112c5fc48d53)
