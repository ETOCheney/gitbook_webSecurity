# SRC漏洞挖掘分享

### 逻辑漏洞

* 用户弱口令
* 用户枚举
* 平行、垂直越权
* 未授权访问 - redis   6379 - mongo 27017 - memcache 11211
* 验证码安全

### 中间件安全

* 弱口令
* 配置缺陷
* 反序列化 - weblogic - jboss - websphere - Jenkins - ……
* 远程代码执行 - jboss
* 文件解析代码执行 - Apache - nginx

## 常见组件

### JAVA系

struts2  
- ognl表达式注入

spring  
- el表达式注入

jboss

tomcat

weblogic

### php系

Thinkphp

Typecho

drupal

joomla

discuz

## 寻找思路

* 确定网站环境（PHP/java/go/python）
* 确定网站中间件（Apache/nignx/tomcat）
* 确定使用组件（cms/framwork/）
* 寻找可控点 用户输入（功能页面） 
* 测试记录观察返回 
* 发现漏洞

### 指纹识别

### 端口扫描

* msscan 
* nmap

### 目录爆破

* dirb
* wwwscan

### 域名遍历

* nmap --script dns-brute
* wydomain

{% hint style="info" %}
字典：从各种扫描工具去手机一些自己觉得好用的字典
{% endhint %}

## 文件上传漏洞

安全校验方式：

### js校验（客户端进行的校验）

* 抓取数据包，更改值
* 更改js代码

### 文件头校验

* PNG  文件头  89 50 4E 47 0D 0A 1A 0A

### content-type字段校验（）

### Apache a0

### 00截断

## 命令注入



