---
title: tornado_process
date: 2019-06-10 00:18:52
tags: tornado web 
categories: python
---

## tornado中使用多进程处理任务

在最近的工作中遇到需要在服务中有重cpu计算的任务

现有两种方案选择

1. celery 实现分布式计算

2. 程序内部实现多进程消费

考虑到轻量级以及响应速度选着后者

原因如下:

1. 该项目中需要传输数据较大，celery在消息处理上消耗较大

2. 搞项目初始化需要加载较多资源，这里不太了解celery是否可以有全局的配置

3. celery 涉及服务组件较多，维护成本较高

## 代码展示

`run_on_executor_decorator` 这个方法只适用于多线程 (这里涉及到进程之间的序列化)

```python

def run_on_executor_decorator(fn):
    executor = kwargs.get("executor", "executor")
    io_loop = kwargs.get("io_loop", "io_loop")

    @functools.wraps(fn)
    def wrapper(self, *args, **kwargs):
        callback = kwargs.pop("callback", None)
        future = getattr(self, executor).submit(fn, self, *args, **kwargs)
        if callback:
            getattr(self, io_loop).add_future(
                future, lambda future: callback(future.result()))
        return future

```

```python
import functools
import os
import time
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor

import tornado.ioloop
import tornado.web


def do_something(*args):
    print(args)
    time.sleep(5)
    print('fafasfasfacscs')


class FutureHandler(tornado.web.RequestHandler):
    executor = ProcessPoolExecutor(10)
    # executor = ThreadPoolExecutor(10)


    @tornado.web.asynchronous
    @tornado.gen.coroutine
    def get(self, *args, **kwargs):

        url = 'www.google.com'

        # 如果是多进程处理 可以直接在这里调用石林方法
        # tornado.ioloop.IOLoop.instance().add_callback((self.do_something))
        self.executor.submit(do_something, url)
        print('works')
        self.finish('It works')

    def do_something(self):
        pass

application = tornado.web.Application([
    (r"/", FutureHandler),
])

if __name__ == "__main__":
    application.listen(7777)
    tornado.ioloop.IOLoop.instance().start()

```

## 参考资料

关于并行计算可以参考

[Python并行编程 中文版](https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/index.html)

## TODO

1. async 的方式如何使用

2. 分布式计算的实现
