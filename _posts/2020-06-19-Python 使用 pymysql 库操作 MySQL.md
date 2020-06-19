---
title:      Python 使用 pymysql 库操作 MySQL
subtitle:   pymysql 库使用
date:       2020-06-19
author:     NSX
catalog: true
tags:
    - 技术
    - 教程

---

# Python 使用 pymysql 库操作 MySQL

# 一 概念

## 1 几个概念

![几个概念](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE1.PNG)



## 2 几个概念间关系

![几个概念间关系](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE2.PNG)

<!-- more -->



## 3 connection 对象（生成对象的方法参数、对象方法）

![connection 对象（生成对象的方法参数、对象方法）](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE2-1.PNG)

![connection 对象（生成对象的方法参数、对象方法）](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE3.PNG)



## 4 cursor 对象（对象方法、fetch 方法）

![cursor 对象（对象方法）](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE4.PNG)

![cursor 对象（对象方法）](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE4-1.PNG)



## 5 一些异常

![一些异常](https://marlous.github.io/2019/04/29/Python-%E4%BD%BF%E7%94%A8-pymysql-%E5%BA%93%E6%93%8D%E4%BD%9C-MySQL/%E5%9B%BE5.PNG)



# 二 编码流程

```
"""
创建一个 connection 对象
"""
db = pymysql.connect(
    host=settings.DB_HOST,
    port=int(settings.DB_PORT),
    user=settings.DB_USER,
    passwd=settings.DB_USER_PASSWORD,
    db=settings.DB
    charset=settings.DB_CHARSET)

try:
    db.ping(reconnect=True)
    cur = db.cursor()
    cur.execute('SHOW DATABASES;')

    """
    使用 cursor 对象的方法获取执行过 MySQL 返回的数据
    fetchall() 是获取多行。返回多个元组，即返回多条记录（rows）,如果没有结果,则返回 ()
    fetchone() 获取一行。返回单个的元组，也就是一条记录（row），如果没有结果，则返回 None
    """
    db_list = list(cur.fetchall())

    """
    补充：使用 connection 对象提交事务（如果是对数据库定义、操作、控制，非查询）
    """
    db.commit()
except Exception as e:
    print(e)
    print("发生错误, 回滚")
    db.rollback()
            
"""
关闭 cursor、connection 对象
"""
cur.close()
db.close()
```

