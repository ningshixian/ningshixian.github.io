---
layout:     post
title:      如何使用Gunicorn启动Flask服务
subtitle:   如何使用Gunicorn启动Flask服务
date:       2019-10-31
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 技术
    - 教程
    - Python
---


# Gunicorn

[Gunicorn ('Green Unicorn')](https://gunicorn.org/) 是一个 UNIX 下的纯 Python WSGI 服务器。

- Gunicorn 启动了被分发到的一个主线程，然后因此产生的子线程就是对应的 worker。
- 主进程的作用是确保 worker 数量与设置中定义的数量相同。因此如果任何一个 worker 挂掉，主线程都可以通过分发它自身而另行启动。
- worker 的角色是处理 HTTP 请求。
- 这个 预 in 预分发 就意味着主线程在处理 HTTP 请求之前就创建了 worker。
- 操作系统的内核就负责处理 worker 进程之间的负载均衡。

它没有其它依赖，容易安装和使用。它所在的位置通常是在反向代理（如 Nginx）或者 负载均衡（如 AWS ELB）和一个 web 应用（比如 Django 或者 Flask）之间



## 关于如何配置 Gunicorn 的实用建议

```python
pip install gunicorn
pip install gevent
```

1. 第一种并发方式（workers 模式，又名 UNIX 进程模式）

   每个 `worker` 都是一个加载 Python 应用程序的 UNIX 进程。`worker`之间没有共享内存。

   建议的 `workers` 数量是 `(2*CPU)+1`。
   对于一个双核（两个CPU）机器，5 就是建议的 `worker` 数量。

   ```python
   gunicorn --workers=5 main:app
   ```

2. 第二种并发方式（多线程）

   Gunicorn 还允许每个 worker 拥有多个线程。在这种场景下，Python 应用程序每个 worker 都会加载一次，同一个 worker 生成的每个线程共享相同的内存空间。

   为了在 Gunicorn 中使用多线程。我们使用了 `threads` 模式。每一次我们使用 `threads` 模式，worker 的类就会是 `gthread`：

   ```
   gunicorn --workers=5 --threads=2 main:app
   ```

   在我们的例子里面最大的并发请求数就是 `worker * 线程`，也就是10。

   在使用 `worker` 和多线程模式时建议的最大并发数量仍然是`(2*CPU)+1`。

3. 第三种并发方式（“伪线程”）

   有一些 Python 库比如（`gevent 和 Asyncio`）可以在 Python 中启用多并发。那是基于协程实现的“伪线程”。

   Gunicrn 允许通过设置对应的 worker 类来使用这些异步 Python 库。

   这里的设置适用于我们想要在单核机器上运行的`gevent`：

   ```
   gunicorn --worker-class=gevent --worker-connections=1000 --workers=3 main:app
   ```

   在这种情况下，最大的并发请求数量是 `3000`。（`3 个 worker * 1000 个连接/worker`）



## 并发 vs. 并行

- 并发是指同时执行 2 个或更多任务，这可能意味着其中只有一个正在处理，而其他的处于暂停状态。
- 并行是指两个或多个任务正在同时执行。

在 Python 中，线程和伪线程都是并发的一种方式，但并不是并行的。但是 workers 是一系列基于并发或者并行的方式。



## 实际案例

通过调整Gunicorn设置，我们希望优化应用程序性能。

1. 如果这个应用是 I/O 受限，通常可以通过使用“伪线程”（gevent 或 asyncio）来得到最佳性能。正如我们了解到的，Gunicorn 通过设置合适的 worker 类 并将 workers数量调整到 (2*CPU)+1 来支持这种编程范式。
2. 如果这个应用是 CPU 受限，那么应用程序处理多少并发请求就并不重要。唯一重要的是并行请求的数量。因为 Python’s GIL，线程和“伪线程”并不能以并行模式执行。实现并行性的唯一方法是增加**workers** 的数量到建议的 (2*CPU)+1，理解到最大的并行请求数量其实就是核心数。
3. 如果不确定应用程序的内存占用，使用 多线程 以及相应的 gthread worker 类 会产生更好的性能，因为应用程序会在每个 worker 上都加载一次，并且在同一个 worker 上运行的每个线程都会共享一些内存，但这需要一些额外的 CPU 消耗。
4. 如果你不知道你自己应该选择什么就从最简单的配置开始，就只是 workers 数量设置为 (2*CPU)+1 并且不用考虑 多线程。从这个点开始，就是所有测试和错误的基准环境。如果瓶颈在内存上，就开始引入多线程。如果瓶颈在 I/O 上，就考虑使用不同的 Python 编程范式。如果瓶颈在 CPU 上，就考虑添加更多内核并且调整 workers 数量。

#### 配置文件 gun.py

```
# gunicorn的配置文件（python文件）：配置文件中的配置会覆盖框架的设置
# 用法：gunicorn [OPTIONS] 模块名：变量名
# 示例：gunicorn -c gun.py flask_slot_newtable:app

bind = '0.0.0.0:8090'      #绑定ip和端口号
workers = 1    #进程数
threads = 5 #指定每个进程开启的线程数
# timeout = 30      #超时
backlog = 2048                #监听队列
worker_class = 'gevent' #使用gevent模式，还可以使用sync 模式，默认的是sync模式
# debug = True
# proc_name = 'gunicorn.proc'
# pidfile = '/tmp/gunicorn.pid'
logfile = '/var/log/gunicorn/debug.log'
loglevel = 'debug' #日志级别，这个日志级别指的是错误日志的级别，而访问日志的级别无法设置
```

#### 执行命令

```
gunicorn -c gun.py xxx:app
```



# 参考

[[译] 通过优化 Gunicorn 配置提高性能](https://juejin.im/post/5ce8cab8e51d4577523f22f8)
