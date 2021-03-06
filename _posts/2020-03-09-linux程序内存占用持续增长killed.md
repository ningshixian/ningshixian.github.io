---
layout:     post
title:      linux程序内存占用持续增长killed
subtitle:   linux程序内存占用持续增长killed
date:       2020-03-09
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 内存占用
    - Linux
    - killed
---

最近在CentOS上运行自己写的程序，程序运行时间久一点就被killed，需要分析原因并找到解决方法. 

首先可能原因是：

1. [内存不够](https://blog.csdn.net/ktigerhero3/article/details/80004315)
2. 程序出错



# 内存不够问题

查看linux 系统日志.

```
vi /var/log/messages
```

如果出现 `kernel: Out of memory: Kill process` 意味着整个系统的内存已经不足，如果不杀死进程的话，就会导致系统的崩溃.

用free命令查看虚拟内存

```
free -h
```

查看某个进程的内存使用情况，使用top命令

```
top -p `pidof rviz`
top -p id
top -u username
```

出现如下情况

> *监控系统报警生产服务进程出现，内存使用率达到 90% 以上*

python程序在长时间(较大负载)运行一段时间后, python 进程的系统占用内存持续升高:

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
14781 root      20   0   10.3g   2.2g   7124 S   0.0 14.5   4:57.13 java
17522 lhadmin   20   0 5915228   1.7g  87264 S   0.0 10.6   1:37.36 python3
#                               ~~~~~~
#                               1.7g 内存占用
```

这里的python进程在经历大量请求处理过程中, 内存持续升高, 直至最终被killed.

<!-- more -->



## 查证过程

昨天有7个record没有返回数据的情况，查日志均是 ==ASR/Slot timeout== 的原因导致。猜测是线程之间发生死锁，gc不能回收，使得多个线程全部挂起，导致内存一直在增长 以及 record服务超时！

具体查证过程如下：

### top 查看 CPU 和内存占用

```
# top
```

### strace 查看系统调用

```
# strace -p 6325
```

### ltrace 查看库函数调用

```
# ltrace -cp 6325
```

### gcore 生成 coredump 文件

为了避免 `gdb attach` 进程造成的其他影响（比如可能出现进程异常退出，死锁突然恢复，影响线上服务等），最好将进程生成一个 coredump 文件，然后再慢慢分析。

```
# gcore 6325
# ls -lsh core.6325
2.7G -rw-r--r-- 1 root root 2.7G 4月  14 00:56 core.6325
```

### gdb 分析 coredump 文件

定位异常：确定python在做什么，是否有大内存消耗任务正在运行，或出现死锁等异常行为

接入gdb

```
# gdb python core.6325
# gdb python2.7 core.6325
$ gdb python <pid>
```

使用 `info threads` 查看当前进程的线程列表，发现大部分都在 `wait` 信号，只有25号线程在做其他事情，切换到25号线程，分析调用栈：

```
(gdb) info threads
  Id   Target Id         Frame
  18   Thread 0x7f561ae6b740 (LWP 9363) 0x00007f561a76520d in poll () at ../sysdeps/unix/syscall-template.S:81
  17   Thread 0x7f5589b17780 (LWP 9463) pthread_cond_wait@@GLIBC_2.3.2 ()
    at ../nptl/sysdeps/unix/sysv/linux/x86_64/pthread_cond_wait.S:185
...
一般加锁、死锁情况存在时，会有线程卡在xx_wait等函数上。
...
如果发现某线程有问题，切换到此线程上
...
(gdb) thread 25
```

查看线程栈信息，

```
(gdb) info stack
```

info stack，这个命令只能查看当前正在运行的某个线程的栈信息

使用原始 `bt` 来分析（添加 `full` 参数可以看更详细的内容）：

```
(gdb) bt full
(gdb) thread apply all bt	# gdb会让所有线程都执行这个命令
```

分析 `Frame # 7` 发现当前线程正在执行 `./service/recall/newuser.py, line 49, in get_gametype_anchor_by_sn` 方法。

找到对应的源代码。通过仔细分析代码，发现在某种情况下确实会出现死循环情况，至此问题解决。



### 死锁解决办法：

1. 尝试用锁加以隔离
2. 取消多线程
3. 手动释放内存：对大的变量存储对象，在使用完成后先del掉，然后在return之前，统一gc.collect()垃圾回收一下



# 总结

1. 遇到线上问题时，优先使用 `gcore PID` 来保存现场
2. 再[使用strace、ltrace](http://rdcqii.hundsun.com/portal/article/597.html)和 `gdb` 分析
3. 如果没有什么线索，可以尝试 [pyrasite-shell](https://pyrasite.readthedocs.io/en/latest/Shell.html) 或 [lptrace](https://github.com/khamidou/lptrace)
4. `gdb`调试 `Python` 进程的时候，运行进程的 `Python` 版本和 python-dbg 一定要匹配



# 参考

[查证微服务死循环导致 CPU 100%问题](https://seealso.cn/debug/endless-loop-causes-cpu-full-problems)

[Python 进程内存增长解决方案](https://zhuanlan.zhihu.com/p/28031057)



# 附录：gdb&tracemalloc

python本身是有垃圾回收的, 但python有如下几种情况可能出现内存泄露（即导致对象无法被回收）:

1. 循环引用
2. 循环引用的链上某个对象定义了`__del__`方法.
3. 对象一直被全局变量所引用, 全局变量生命周期长.
4. 垃圾回收机被禁用或者设置成debug状态, 垃圾回收的内存不会被释放.
5. 引用对象未释放（数据库连接等）



**gdb安装教程**

详细信息可以参考 [debug-with-gdb](https://wiki.python.org/moin/DebuggingWithGdb)

```
touch /etc/yum.repos.d/CentOS-Debuginfo.repo
vi /etc/yum.repos.d/CentOS-Debuginfo.repo
```

添加以下内容至文件中：

```
# CentOS-Debug.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#

# All debug packages from all the various CentOS-5 releases
# are merged into a single repo, split by BaseArch
#
# Note: packages in the debuginfo repo are currently not signed
#

[debug]
name=CentOS-7 - Debuginfo
baseurl=http://debuginfo.centos.org/7/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-Debug-7
enabled=1
```

安装依赖

```
sudo yum install yum-utils
sudo debuginfo-install glibc
sudo yum install gdb python-debuginfo
```

完成！



**查看python内存泄露的工具**

- tracemalloc --- 跟踪内存分配: 究极强, 可以直接看到哪个(哪些)对象占用了最大的空间（内存块）, 这些对象是谁, 调用栈是啥样的, python3直接内置, python2如果安装的话需要编译. 它提供以下信息：

  - Traceback where an object was allocated
  - 每个文件名和每行号分配的内存块的统计信息：分配的内存块的总大小，数量和平均大小
  - 计算两个快照之间的差异以检测内存泄漏

  - 显示前10项

  ```
  import tracemalloc
  
  tracemalloc.start()
  
  # ... run your application ...
  
  snapshot = tracemalloc.take_snapshot()
  top_stats = snapshot.statistics('lineno')
  
  print("[ Top 10 ]")
  for stat in top_stats[:10]:
      print(stat)
  ```

  - 计算差异

  ```
  import tracemalloc
  tracemalloc.start()
  # ... start your application ...
  
  snapshot1 = tracemalloc.take_snapshot()
  # ... call the function leaking memory ...
  snapshot2 = tracemalloc.take_snapshot()
  
  top_stats = snapshot2.compare_to(snapshot1, 'lineno')
  
  print("[ Top 10 differences ]")
  for stat in top_stats[:10]:
      print(stat)
  ```

  - 显示最大内存块的回溯的代码：

  ```
  import tracemalloc
  
  # Store 25 frames
  tracemalloc.start(25)
  
  # ... run your application ...
  
  snapshot = tracemalloc.take_snapshot()
  top_stats = snapshot.statistics('traceback')
  
  # pick the biggest memory block
  stat = top_stats[0]
  print("%s memory blocks: %.1f KiB" % (stat.count, stat.size / 1024))
  for line in stat.traceback.format():
      print(line)
  ```

- pyrasite: 牛逼的第三方库, 可以渗透进入正在运行的python进程动态修改里边的数据和代码(其实修改代码就是通过修改数据实现)

