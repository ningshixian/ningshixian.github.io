## 虚拟环境 virtualenv

>  [virtualenv](http://pypi.python.org/pypi/virtualenv) 是一个创建独立的 Python 环境
>
>  https://learnku.com/docs/python-guide/2018/virtualenvs-lower-level-virtualenv/3257
>
>  https://pythonguidecn.readthedocs.io/zh/latest/writing/structure.html

通过 pip 安装 virtualenv ：

```
$ pip install virtualenv
```

为项目创建一个虚拟环境，名叫`my_project`：

```
$ cd my_project_folder
$ virtualenv my_project

运行  virtualenv 带上选项  --no-site-packages  将不会包含已经全局安装的包。这样有助于保持包列表的整洁以防万一之后需要访问它。
[ 这在 virtualenv 1.7 是默认的哦。 ]
```

开始使用虚拟环境前，需要先激活：

```
$ source my_project/bin/activate
$ my_project\Scripts\activate
```

安装包的话就与往常一样，如：

```php
$ pip install XXX
```

如果你在虚拟环境中暂时完成了工作，可以这样停用它：

```php
$ deactivate
```

为了保持环境的一致性，“冻结” 当前环境包的状态是正确的选择：

```
$ pip freeze > requirements.txt
```

该命令将创建一个 requirements.txt 文件，里面包含有当前环境所有包的简单列表及对应的版本，这样就能完全搭建出与之前一致的环境了：

```
$ pip install -r requirements.txt
```

这样有助于在跨设备，跨部署，跨人员的情况下保证环境的一致性。

