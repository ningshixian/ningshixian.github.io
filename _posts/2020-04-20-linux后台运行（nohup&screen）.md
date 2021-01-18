---
layout:     post                    # 使用的布局（不需要改）
title:      linux后台运行（nohup&screen）              # 标题 
subtitle:   nohup&screen            # 副标题
date:       2020-04-20              # 时间
author:     NSX                     # 作者
header-img: img/post-bg-2015.jpg    # 这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               # 标签
    - 技术
    - 教程
    - Linux
---

# linux后台运行（nohup&screen）

## nohup

```
nohup python -u flask_test.py > log.txt 2>&1 &
```

为避免python的输出缓冲，将程序中print的内容写入日志，使用-u参数，使得python不启用缓冲.



## screen

screen 是一个非常有用的命令，提供从单个 SSH 会话中使用多个 shell 窗口（会话）的能力。当会话被分离或网络中断时，screen 会话中启动的进程仍将运行，你可以随时重新连接到 screen 会话

安装Screen

```
yum -y install screen
apt-get -y install screen
```



```
创建窗口
screen -S intent
python xxx.py

查看哪些窗口正在运行
screen -ls
screen -list
screen -r ex 程序执行命令

恢复会话窗口
screen -r <线程ID>/<作业名称> 

会话分离
ctrl+a 然后 d

杀死会话窗口
kill -9 线程号
screen -wipe

查看手册
man screen
```

