---
description: 介绍几款常用的自动化漏洞扫描工具
---

# 漏洞扫描工具

## Nessus

> Nessus 是目前全世界最多人使用的系统漏洞扫描与分析软件。总共有超过75000个机构使用Nessus作为扫描该机构电脑系统的软件。

### 软件特色

* 提供完整的电脑漏洞扫描服务，并随时更新其漏洞数据库。 
* 不同于传统的漏洞扫描软件，Nessus 可同时在本机或远端上摇控，进行系统的漏洞分析扫描。 
* 其运作效能能随着系统的资源而自行调整。如果将主机加入更多的资源（例如加快CPU速度或增加内存大小），其效率表现可因为丰富资源而提高。 
* 可自行定义插件（Plug-in） 
* NASL（Nessus Attack Scripting Language）是由Tenable所开发出的语言，用来写入Nessus的安全测试选项。 
* 完整支持SSL（Secure Socket Layer）。 
* 自从1998年开发至今已谕十年，故为一架构成熟的软件。

### 下载及安装过程

#### 申请试用激活码

打开此链接[https://www.tenable.com/products/nessus/activation-code](https://www.tenable.com/products/nessus/activation-code)，然后选择try for free

![](../.gitbook/assets/image%20%28147%29.png)

填写信息，然后会给你一个使用7天的激活码

![](../.gitbook/assets/image%20%2825%29.png)

#### 下载 安装包并安装

打开下载链接[https://www.tenable.com/downloads/nessus\#download?utm\_campaign=00000292&utm\_promoter=tenable-dm&utm\_medium=email&utm\_content=confirmation&utm\_source=nessus-trial](https://www.tenable.com/downloads/nessus#download?utm_campaign=00000292&utm_promoter=tenable-dm&utm_medium=email&utm_content=confirmation&utm_source=nessus-trial)

下载对应的版本，由于我是kali 2018 64位，所以我下载如图版本

![](../.gitbook/assets/image%20%2820%29.png)

将安装文件复制到虚拟机，执行dpkg -i 下载的文件

```text
dpkg -i Nessus-8.0.1-debian6_amd64.deb
```

启动Nessus 服务

```text
/etc/init.d/nessusd start
```

查看状态

```text
/etc/init.d/nessusd status
```

浏览器访问 https://虚拟机ip:8834，设置用户名密码

![](../.gitbook/assets/image%20%2865%29.png)

填写激活码

![](../.gitbook/assets/image%20%2829%29.png)



