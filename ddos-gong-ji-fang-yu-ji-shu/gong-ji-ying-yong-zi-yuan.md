---
description: DDos攻击还可以攻击应用资源
---

# 攻击应用资源

> 近年来，消耗应用资源的分布式拒绝服务攻击正逐渐成为拒绝服务攻击的主要手段之一，而由于DNS和Web服务的广泛性和重要性，这两种服务也就成为了消耗应用资源的分布式拒绝服务攻击的最主要的攻击目标。

## DNS QUERY 攻击

dns查询过程

1. 客户端向本地DNS服务器查询www.baidu.com，本地DNS无缓存 
2. 本地向根DNS服务器查询.com，根服务器无缓存 
3. 根服务器返回查询结果.com 
4. 本地DNS向顶级域名服务器查询baidu.com，顶级域名无缓存 
5. 顶级域名服务器返回baidu.com 
6. 本地DNS向权威DNS查询www.baidu.com 
7. 权威DNS服务器返回www.baidu.com 
8. 客户端访问www.baidu.com的真实IP地址

![DNS&#x67E5;&#x8BE2;&#x8FC7;&#x7A0B;](../.gitbook/assets/image%20%28129%29.png)

DNS Flood 正是利用客户端的一次查询，服务器在未命中时需要多次查询来消耗DNS服务器资源。

![DNS Flood &#x653B;&#x51FB;DNS&#x670D;&#x52A1;&#x5668;](../.gitbook/assets/image%20%2841%29.png)

## CC攻击（HTTP-Flood）

> CC攻击（Challenge Collapsar）CC攻击的本名叫做HTTP-FLOOD，是一种专门针对于Web的应用层FLOOD攻击，攻击者操纵网络上的肉鸡，对目标Web服务器进行海量http request攻击。

**CC攻击发起的是合法的请求**，比如去论坛上读一个贴子，服务器会先去查询你是否有权限访问该贴，然后再从数据库里读出该贴。这里至少会访问两次数据库。如果数据库优化没有做好，这时服务器消耗大量资源在数据库查询上。

![CC attack](../.gitbook/assets/image%20%2861%29.png)

## slow HTTP Dos\(HTTP慢速攻击\)

> Slow HTTP Dos AttACKs（慢速HTTP拒绝服务攻击）

{% hint style="warning" %}
HTTP慢速攻击对基于线程处理的Web服务器影响显著，如apache、dhttpd，

而对基于事件处理的Web服务器影响不大，如nginx、lighttpd。
{% endhint %}

攻击者在发送HTTPPOST请求时，**在请求头部中将Content-Length设置为一个很大的值，并将HTTPBODY以非常缓慢的速度一个字节一个字节的向Web服务器发送**，这样攻击者会长时间占有这个HTTP连接。

![HTTP &#x6162;&#x901F;&#x653B;&#x51FB;](../.gitbook/assets/image%20%2868%29.png)

## Slowloris Attack（HTTP 慢速攻击）

HTTP协议规定请求头以一个空行结束，所以完整的http请求头结尾是 \r\n\r\n。 **使用非正常的\r\n来结尾，就会导致服务端认为我们的请求头还没结束**，等待我们继续发送数据直到超时时间。

![](../.gitbook/assets/image%20%285%29.png)



