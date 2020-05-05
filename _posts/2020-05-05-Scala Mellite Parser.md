---
title:      Scala Mellite Parser
subtitle:   Scala Mellite Parser
date:       2020-05-05
author:     NSX
catalog: true
tags:
    - 技术

---

# Scala Mellite Parser


1. git clone 拉取远程分支到本地

    新项目地址：(Mellite和MagusLibrary这两个项目都要pull)

    ```
    git clone -b mellite_dev http://git.longhu.net/Magus/Mellite.git
    git clone ssh://git@git.longhu.net:8010/Magus/MagusLibrary.git
    ```

    旧项目地址：

    ```
    git clone -b ssh://git@git.longhu.net:8010/Magus/Djinn.git
    git clone ssh://git@git.longhu.net:8010/Magus/Calibur.git
    ```

    

2. git pull拉取远程分支到本地

    ```
    git pull <远程主机名> <远程分支名>:<本地分支名>`
    git pull origin master:wy
    ```

    

3. 若本地和远程仓库有冲突

    ```
    git rebase -i  [startpoint]  [endpoint]
    或者：
    VCS→git→pull→选择rebase策略
    ```

    

4. 解决win下IDEA的编码问题

    ```
    Help->Edit Custom VM Option	-> 最后添加以下代码：
    
    -Dconsole.encoding=UTF-8
    -Dfile.encoding=UTF-8
    
    Settings→Editor→File Encodings→add Path
    ```

    PS: 保证Scala插件正常运行

    

5. Run -> Run -> 启动 Scala Console

    

6. 测试Parser代码

    ```
    import magus.djinn.parsing.ops.XMLOps._
    import magus.zephyr.data.MegaPok
    import magus.puck.utils.json.JsonUtils._
    
    val nl = "南京公司非操盘的权重项目本年度实际签约货值累计达到了多少？"
    
    <root>
    <file name="QueryForm1" />
    </root>.parse(nl).as[MegaPok].printTree
    
    <root>
    <file name="QueryForm1" />
    </root>.parse(nl).as[MegaPok].toJson.prettyPrint
    
    ```



7. 报错排查

   ```
   1、无条件输出解析结果
   
   val p2 = QueryParser("QueryForm1UPM")
   p2.parse(nl).as[MegaPok].printTree
   
   
   2、测试某指标是否能够正确解析
   
   <root><file name="select/Measure"/></root>.parse("全年定稿预算签约货值").as[MegaPok].printTree
   ```

   快捷键运行：ctrl + 回车