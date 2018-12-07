# Bypass UAC

### Bypass UAC <a id="csaa-11-12-bypass-uac"></a>

### 【实验目的】 <a id="-"></a>

  通过本实验了解Bypass uac思路

### 【实验环境】 <a id="-"></a>

  工具：VS或Dev C++、Process Explorer ​ 系统：win10

### 【实验原理】 <a id="-"></a>

  无

### 【实验步骤】 <a id="-"></a>

**1、观察 计算机\mscfile\shell\open\command中的内容**

（1）用管理员权限打开注册表。

![mark](http://wfwewefw.yuxuni.cn/blog/180925/L88Ha049i5.png?imageslim)

（2）可以看到注册表界面。

观察 计算机\mscfile\shell\open\command中的内容

![mark](http://wfwewefw.yuxuni.cn/blog/180925/JiF7HHK4kc.png?imageslim)

（3）用管理员身份打开Process Explorer。

```text
Process Explorer是由Sysinternals开发的**Windows系统和应用程序监视**工具，目前已并入微软旗下。不仅结合了Filemon（文件监视器）和Regmon（注册表监视器）两个工具的功能，还增加了多项重要的增强功能。包括稳定性和性能改进、强大的过滤选项、修正的进程树对话框（增加了进程存活时间图表）、可根据点击位置变换的右击菜单过滤条目、集成带源代码存储的堆栈跟踪对话框、更快的堆栈跟踪、可在 64位 Windows 上加载 32位 日志文件的能力、监视映像（DLL和内核模式驱动程序）加载、系统引导时记录所有操作等。
```

![mark](http://wfwewefw.yuxuni.cn/blog/180925/giiG6lAECh.png?imageslim)

Process Explorer界面如下。

![mark](http://wfwewefw.yuxuni.cn/blog/180925/k5d8lId9bH.png?imageslim)

**2、尝试运行eventvwr.exe，观察被启动的进程**

![mark](http://wfwewefw.yuxuni.cn/blog/180925/b28831KBia.png?imageslim)

![mark](http://wfwewefw.yuxuni.cn/blog/180925/DbmI9FeI7k.png?imageslim)

![mark](http://wfwewefw.yuxuni.cn/blog/180925/cGjmJc50i2.png?imageslim)

![mark](http://wfwewefw.yuxuni.cn/blog/180925/2EJi9EgH5L.png?imageslim)

杀掉mmc.exe 判断是不是一个程序。

![mark](http://wfwewefw.yuxuni.cn/blog/180925/GK6C5ELmFc.png?imageslim)

**3、人为创建注册表字段，并在其中加入键值，尝试让eventvwr.exe去执行其他exe程序**

![mark](http://wfwewefw.yuxuni.cn/blog/180925/L54CHH0k3c.png?imageslim)

**4、尝试运行eventvwr.exe，利用Process Explorer查看powershell.exe是否被启动，以及powershell.exe的权限等级**

![mark](http://wfwewefw.yuxuni.cn/blog/180925/C5biDkGF03.png?imageslim)

![mark](http://wfwewefw.yuxuni.cn/blog/180925/f4K3C69f2J.png?imageslim)

**5、调用Windows API来修改注册表值**

低权限下执行UACBypass.exe

这个漏洞利用程序相当于上一步手动添加注册表的效果，调用Windows API来修改注册表值。通过这种方式我们可以Bypass UAC。

![mark](http://wfwewefw.yuxuni.cn/blog/180925/fJJk4ljH21.png?imageslim)

执行成功

![mark](http://wfwewefw.yuxuni.cn/blog/180925/c3CicJh9La.png?imageslim)

**6、再次尝试运行eventvwr.exe，利用Process**

![mark](http://wfwewefw.yuxuni.cn/blog/180925/cGjmJc50i2.png?imageslim)

**7、再次尝试运行 eventvwr.exe，利用Process Explorer查看powershell.exe是否被启动，以及powershell.exe的权限等级**

可以看到执行我们的powershell.exe 程序

![mark](http://wfwewefw.yuxuni.cn/blog/180925/j4I1eBikll.png?imageslim)

