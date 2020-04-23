---
layout:     post
title:      通过Python来操作kylin数据
subtitle:   kylinpy工具库使用
date:       2020-04-01
author:     NSX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 技术
    - 教程
    - Python
---

# 通过Python来操作kylin数据

1. 安装依赖的包（py2/py3都支持)

```
pip install kylinpy
pip install sqlalchemy
pip install --upgrade kylinpy
```

 kylinpy工具库包含两个可使用原件. 想要了解更多关于此工具库信息请点击[Github仓库](https://github.com/Kyligence/kylinpy).

- Apache Kylin 命令行工具
- Apache Kylin SQLAchemy方言

<!-- more -->



2. 示例代码

```
#!/usr/bin/env python
# coding=utf-8

import sqlalchemy as sa
import kylinpy
import pymysql


# SQLAlchemy 实例
def kylin_query1(conn_str, query_sql):
    kylin_engine = sa.create_engine(conn_str)
    print(kylin_engine.table_names())
    results = kylin_engine.execute(query_sql)
    print([e for e in results])
   
# kylinpy 实例
def kylin_query1(query_sql):
    kylin = kylinpy.KylinCluster(host=ip, username="ADMIN", password="", project="")
    # print(kylin.projects())
    results = kylin.query(query_sql)
    print(results["results"])

if __name__ == "__main__":
    conn_str = "kylin://<username>:<password>@<ip>:<port>/<project>?version=<v1|v2>&prefix=</kylin/api>"
    query_sql = "select  userid, datetime, count(*) c from soda_report group by userid, datetime"
    
    kylin_query1(conn_str, query_sql)    
    kylin_query2(query_sql)
```



# 参考

[通过Python来操作kylin](https://www.cnblogs.com/654wangzai321/p/9869939.html)

[Kylin Python 客户端工具库 - kylinpy](http://kylin.apache.org/cn/docs21/tutorial/kylin_client_tool.html)