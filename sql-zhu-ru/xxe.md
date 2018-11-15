---
description: >-
  如今越来越多主要的web程序被发现和报告存在XXE（XML External Entity
  attack）漏洞，比如说Facebook、Paypal等等，尽管XXE漏洞已经存在了很多年，但是它从来没有获得它应得的关注度。很多XML的解析器默认是含有XXE漏洞的，意味着开发人员有责任确保这些程序不受此漏洞的影响。XXE全称XML外部实体注入（XML
  External Entity）形成的原因大都是
---

# XXE

## XML

> XML用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型XML是一种允许用户对自己的标记语言进行定义的源语言。

### 结构

XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素。

![](../.gitbook/assets/image%20%28132%29.png)

通过&来调取变量

```markup
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE xxe [
<!ELEMENT name ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<root>
<name>&xxe;</name>
</root>
```

## 常见危害

### 窃取敏感信息

如果有回显直接读取文件

例如 

