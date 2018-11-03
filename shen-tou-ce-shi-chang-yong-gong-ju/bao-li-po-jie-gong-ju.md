# 暴力破解工具

## Hydra

> hydra（九头蛇）是一个暴力破解工具，可运行在任意平台之上，可爆破多达20多种协议的口令。

官网：[http://www.thc.org/thc-hydra](http://www.thc.org/thc-hydra)

支持破解：



## Hydra参数解释

* -R 根据上一次进度继续破解
* -S 使用SSL协议连接 
* -s 指定端口
* -l  指定用户名
* -L  指定用户名字典（文件） 
* -p  指定密码破解 
* -P  指定密码字典（文件） 
* -e  空密码探测和指定用户密码探测（ns） 
* -c  用户名可以用:分割（useznane:paseword）可以代替-l usernane -p passvord 
* -o  输出文件
* -t  指定多线程数量，默认为16个线程 
* -vV 显示详细过程 
* server 目标IP 
* service 指定服务名（telnet ftp pop3assqlnysqlash ssh2......）

