---
layout:     post
title:      VSCode IDE使用指南
subtitle:   VSCode IDE使用指南
date:       2019-08-30
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 教程
    - VSCode
---

# VSCode IDE使用指南

**Visual Studio Code**（简称**VS Code**）是一个由微软开发，同时支持Windows 、 Linux和macOS等操作系统且开放源代码的代码编辑器

![img](https://mmbiz.qpic.cn/mmbiz_png/D67peceibeISArWuibmhjJoicoEvFFK3ZeLbxeFQNJsmFjHsZWG5MjyY2M3Rd9gDNwiadCXK6tczPajRL5k0Mq3VSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 如何配置VSCode
- 插件推荐
- 利用VS Code进行远程开发

<!-- more -->



## 如何配置VSCode

1. **安装VSCode的Python支持**
2. **设置字体和字号**
3. 插件推荐



## 插件推荐

我最喜欢的是它的插件能力，几乎想要啥功能都能找到插件支持，应该不用我安利你们都会喜欢的。工欲善其事必先利其器，开发前先把所以提升效率的利器搭好会让今后慢慢的编程长路舒服很多！

- `Settings Sync` 

    这个插件可以实现同步你的vscode设置，包括setting文件，插件设置等

- `Python`  

    让VS Code支持python编程,提供了代码分析，高亮，规范化等很多基本功能

- `Material Theme`  

    VSCode主题（多种）

- `vscode-icons`  

    改变vscode原有的icon(颜控)

- `filesize`  

    一款在左下角显示文件大小的插件

- `Python Indent` 

    在每次换行的时候会解决Python Indentation的一些问题

- `indent-rainbow`

  提示代码缩进是否到位

- `Code Runner` | `Output Colorizer` 

    右键里有在终端中直接运行Python文件，并对输出给与颜色标注，方便观看

- `Chinese (Simplified) Language Pack for Visual Studio Code` 

    适用于 VS Code 的中文（简体）语言包

- `autoDocstring`  
  
  自动的把所有输出输出参数以一定的风格放到函数或者类注释里面，实乃神器 
  
  Press enter after opening docstring with triple quotes (""" or ''')
  
- `Python Preview`

    展示Python中内置变量的值，方便有时候调试展示值。

- `flake8`

  代码错误提示 linting

- `black`
  
  代码自动格式化（通过命令 `black xxx.py`）

- `Beautify`

  同上，美化代码利器，方便他人阅读

- `Pylance`

    **微软推出**的代码补全工具，提供的一些关键功能包括有：类型信息、自动导入、类型检查诊断和多根工作区支持

    其他工具：还有`aiXcoder` 北大出品（不太好用）、`kite` 实时补全代码、`TabNine` 「程序员的杀手级应用」（cpu 消耗大，笨重）

- `Visual Studio IntelliCode`

  AI协助开发工具

- `Polacode`

  优雅的分享代码截图，可以把代码保存成美观的图片

- 远程连接相关插件



## 利用VS Code进行远程开发

微软在 PyCon 2019 大会上发布了VS Code Remote ，从 1.35.0 版本正式提供可以在本地编辑远程开发环境的文件的功能。

VS Code 使用远程开发插件，不论在哪里，只要有电脑都能方便的打开云端开发环境，非常的方便，是一个大幅提升生产力的好工具！

### 远程开发配置

**配置SSH环境变量**

由于Git自带SSH客户端程序，如果你还没装Git的话，这里要先安装 Git，所以配置 Git 的 bin目录到环境变量的 PATH 变量下，这样VS Code连接的时候就能找到它了。

**安装远程开发插件**

要能连上远程主机，首先我们需要下载VS Code远程开发插件，VS Code其实是提供了一个远程开发插件包，包括：

- Remote - SSH - 通过使用 SSH 链接虚拟或者实体Linux主机。
- Remote - Containers – 连接 Docker 开发容器。
- Remote - WSL - 连接 Windows Subsystem for Linux （Linux子系统）。

打开软件的扩展界面，搜索 `Remote` 开头的插件，也能看到这三个的不同远程开发插件，**我们这里连接的是云主机，选择安装 Remote - SSH 插件安装即可。**

**配置远程连接**

1. 首先点侧边栏的「远程资源管理器」之后点击「设置按钮」，进入远程机器配置界面。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/ceNmtYOhbMRCYaSbicLwbVopKvMTVnRI2lvb7xZuWykUb9qhPGHWa2fQibich6VCo6GrY94PSd2qN3DsUzSm24dEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   

2. 修改 ssh 配置文件，用于登录远程机器，各项含义在图中有说明。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/ceNmtYOhbMRCYaSbicLwbVopKvMTVnRI2h8icWGwx8E3kYk4ZhWDWbWntEOsgniacbiae3olepAWhyUzeDeliaYfdLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   

3. 点击连接，登录远程服务器，需要输入几次远程服务器的密码（后面会教你怎么免密登录），输入确认即可。第一次连接会做VS Code Server的初始化工作比较慢，耐心等待。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/ceNmtYOhbMRCYaSbicLwbVopKvMTVnRI2fRVqjmjMrHhptOArRqE52iag7ZmegeSj4tl8Yqo3QQjNa4ibH7gFNBAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



4. 登录成功，即可像操作本地环境一样，在VS Code客户端操作远程云主机上的文件。注意，**下图中的「打开文件夹」已经是远端机器上的目录结构了。**

   ![img](https://mmbiz.qpic.cn/mmbiz_png/ceNmtYOhbMRCYaSbicLwbVopKvMTVnRI2rAwVZSAHBc0AQZwwWcKJDuOHtRmPLKkcaMw0DlSfrqNYUzWRXS7GmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



5. 给远程VS Code 安装插件。安装的插件是在云服务器的VS Code上，对本机的VS Code没有影响，插件在远端提供功能，比如代码审查、自动补齐等等，而这所有的一切就像在本地操作一样，对文件的更改也是直接操作的云主机上的文件，丝滑连接。

   

6. 代码编辑与远程终端调试。打开文件编辑的是云服务器的文件，同时可以打开云服务终端，直接在终端操作编译或者查看云服务器信息。

   ![img](https://mmbiz.qpic.cn/mmbiz_png/ceNmtYOhbMRCYaSbicLwbVopKvMTVnRI27iaqtfZDPUjshvVlSBsAnl3Y5JUFhRJ7QM7AHBavXaTMNldr6TvahaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### SSH免密登录配置

按照上面的配置步骤，每次连接到远程服务器，都需要输入服务器登录密码很麻烦，可以配置SSH免密登录，免去每次输入密码的烦恼，具体操作步骤如下：

- 打开win cmd终端，输入 ssh-keygen -t rsa 生成秘钥对

- 打开生成的秘钥保存路径，拷贝 `id_rsa.pub` 内容，添加到到云服务器的 `~/.ssh/authorized_keys` 文件后面。
- 尝试再次连接，不用输密码了，enjoy！



### 常见问题

- [vscode中无法激活conda虚拟环境](https://blog.csdn.net/random0708/article/details/105988932)




# 参考

- [工具篇-vscode效率提升插件](https://zhuanlan.zhihu.com/p/73452541)

- [新版 Kite：实时补全代码，Python 之父都发声力挺！](https://www.leiphone.com/news/201911/ESeY1uZvYRdP5V43.html)

- [利用VS Code进行远程开发，就问你香不香？](https://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247494803&idx=3&sn=03608ca1e275414bba356780ebaa86a8&chksm=fa0d8312cd7a0a041d550e20e9b1e9c2be1e87ba23a71a15e73c9eec5f999b1b9cee5782efb8&scene=126&sessionid=1604029083&key=a6a963ce8dc9af21e4278ddbc10ce6143fb180b87c87924bd6cc36ac923408a2c26d85e0e7ba3e2857137a15559d2599e10cc8d7d51a6c5fc2e735397d01d68ccf310f661fd42bb4c02bd91705c76bebcafa63c44341883589c59f26a988bb9c25db32e946bb4f7ee273d5327e572969a3c4325bfa607682c6816ea92b10def7&ascene=1&uin=MjM2MDA1NjcyMQ%3D%3D&devicetype=Windows+10+x64&version=6300002f&lang=zh_CN&exportkey=A6uPvof82tWGpt6NxM5SR%2F4%3D&pass_ticket=r5dszQk2lRNNteX%2BZmX6%2B1wF2g3D57o1FKmEWEXG6IdPb9qLFiUO6rf4TKQjjajJ&wx_header=0)

- 