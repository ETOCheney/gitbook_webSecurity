---
description: 整理顺便复习数据库知识
---

# 数据库基础补漏

## 主流数据库简介

#### Microsoft SQL Server 

SQL Server 是Microsoft公司推出的关系型数据库管理系统 具有使用方便可伸缩性好与相关软件集成程度高等优点 从旧版本的个人电脑到运行Microsoft Windows server的大型多处理器的服务 器都可以使用

#### MySQL

 MySQL是现在非常流行的的关系型数据库管理系统，在WEB应用方面MySQL是最好的RDBMS（Relational Database Management System：关系数据库管理系统）应用软件之一。

1.命令行下连接：

`[root@host]#mysql-u root-p`

2.PHP连接：

`mysqli_connect(host,username,password,dbname,port,socket);`

#### Oracle

 Oracle Database，又名0racle RDBMS，或简称Oracle。 甲骨文公司的一款关系数据库管理系统。 在数据库领域一直处于领先地位的产品，在大公司的大型网络中运用非常多。

#### PostgreSQL

 PostgreSQL是一个功能强大的开源对象关系数据库管理系统（ORDBMS）稳定性极强，用于安全地存储数据

## 数据库识别方法

### 盲跟踪：

* web应用技术
* 不同数据库SQL语句差异

### 非盲跟踪：

* 报错、直接查询

### 利用字符串链接的方式去匹配数据库产品

<table>
  <thead>
    <tr>
      <th style="text-align:left">数据库服务器</th>
      <th style="text-align:left">查询</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Microsoft SQL Server</td>
      <td style="text-align:left">SELECT ‘str1’+'str2' //sql server 字符串拼接用+号</td>
    </tr>
    <tr>
      <td style="text-align:left">MySQL</td>
      <td style="text-align:left">
        <p>SELECT 'str1' 'str2' //mysql字符串拼接用空格</p>
        <p></p>
        <p>SELECT CONCAT('str1','str2')</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Oracle</td>
      <td style="text-align:left">
        <p>SELECT 'str1'||'str2' //oracle数据库字符串拼接用|</p>
        <p></p>
        <p>SELECT CONCAT('str1','str2')</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">PostgreSQL</td>
      <td style="text-align:left">
        <p>SELECT 'str1'||'str2'</p>
        <p></p>
        <p>SELECT CONCAT('str1','str2')</p>
      </td>
    </tr>
  </tbody>
</table>### 利用特定数据库函数匹配数据库产品

<table>
  <thead>
    <tr>
      <th style="text-align:left">数据库服务器</th>
      <th style="text-align:left">查询</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Microsoft SQL Server</td>
      <td style="text-align:left">
        <p>@@pack_received 返回 Microsoft® SQL Server™ 自上次启动后从网络上读取的输入数据包数目。</p>
        <p>@@rowcount 返回受上一语句影响的行数。 如果行数大于 20 亿，请使用 <a href="http://msdn.microsoft.com/zh-cn/library/ms181406.aspx">ROWCOUNT_BIG</a>。</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">MySQL</td>
      <td style="text-align:left">
        <p>connection_id() 链接ID</p>
        <p>last_insert_id()</p>
        <p>row_count() 返回前一个SQL进行UPDATE，DELETE，INSERT操作所影响的行数</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Oracle</td>
      <td style="text-align:left">BITAND(1,1) 返回两个数值型数值在按位进行AND 运算后的结果。</td>
    </tr>
    <tr>
      <td style="text-align:left">PostgrcSQL</td>
      <td style="text-align:left">SELECT EXTRACT(DOW FROM NOWO) 格式化时间函数</td>
    </tr>
  </tbody>
</table>### 数据库版本查询

| 数据库 | 查询 |
| :--- | :--- |
| Mssql    | select @@version |
| MySQL |  select version\(\)/select @@version  |
| Oracle | select banner from $version |
| Postgresql  | select version\(\) |

### 数据库在字符串处理时的区别

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left">Ms SQL</th>
      <th style="text-align:left">My SQL</th>
      <th style="text-align:left">Access</th>
      <th style="text-align:left">Oracle</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">长度</td>
      <td style="text-align:left">len('abc')=3</td>
      <td style="text-align:left">length('abc')=3</td>
      <td style="text-align:left">len('abc')=3</td>
      <td style="text-align:left">length('abc')=3</td>
    </tr>
    <tr>
      <td style="text-align:left">截取左右</td>
      <td style="text-align:left">
        <p>left('abc',2)='ab'</p>
        <p>right('abc',2)='bc'</p>
      </td>
      <td style="text-align:left">
        <p>left('abc',2)='ab'</p>
        <p>right('abc',2)='bc'</p>
      </td>
      <td style="text-align:left">
        <p>left('abc',2)='ab'</p>
        <p>right('abc',2)='bc'</p>
      </td>
      <td style="text-align:left">用substr代替</td>
    </tr>
    <tr>
      <td style="text-align:left">截取中间</td>
      <td style="text-align:left">substring('abc',2,1)='b'</td>
      <td style="text-align:left">
        <p>substring('abc',2,1)='b'</p>
        <p>mid('abc',2,1)='b'</p>
      </td>
      <td style="text-align:left">mid('abc',2,1)='b'</td>
      <td style="text-align:left">substr('abc',2,1)='b'</td>
    </tr>
  </tbody>
</table>## SQL语法

C=create创建

`CREATE DATABASE testdb;`

`CREATE TABLE table name(column name column_type);`

U=Update 更改 

`UPDATE table name SET field1=new-value1，field2=new-value2 [WHERE Clause]`

 R=Retrieve读取 

`SELECT column name，column name FROM table name [WHERE Clause] [OFFSET M JILIMIT N]` 

D=Delete 删除 

`DELETE FROM table_name[WHERE Clause]`



排序order by

`SELECTFROM test table ORDER BY userid;`

order by：要是后面跟着的数子超出子段数的时候，则会报错！遇过这个可以确定字段数。

分组 group by 

`SELECT name,COUNT`\(\*\)`FROM test table GROUP BY name;`

限定条数limit 

`SELECTFROM test table limit 0,10;`

`SELECT FROM test table limit 1,5;`

select\*from table limit m，n其中m是指记录开始的index，从0开始，表示第一条记录n是指从第m+1条开始，取n条

组合使用

 `SELECT*FROM test table LIMIT 0,5 ORDER BY userid;`

### 联合查询

#### 基础用法

联合查询可以将两次查询的结果拼接到一个表中例如：

![](../.gitbook/assets/image%20%2882%29.png)

union会自动去除重复的值，想要全部显示则需要使用union all

![](../.gitbook/assets/image%20%2880%29.png)

#### 利用union猜测列数

因为查询语句构造问题，可直接否认掉之前的查询，执行一个全新的语句来执行，需要注意的是查询的列应当和之前对应。例如：

```text
select 1,2,3 where 1=2 union select 4,5,6
```

还可以利用union来猜测列数，用and union select 1，2，3，4，5，6...；来猜解列数（字段数），只有列数相等了，才能返回True

#### 结合exists（）函数猜解表名 

```text
and exists(selec…)
```

exists（）函数用于检查子查询是否至少会返回一行数据。此时不返回任何数据，而是返回True或者False，用来判断是否存在表。

#### 结合系统函数暴数据库信息

在MySQL中，把information\_schema 看作是一个数据库，确切说是信息数据库。 其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权限等。

我们可以借助information\_Schema数据库来浏览数据库信息，例如Mysql

查看所有数据库：

```sql
select * from information_schema.schemata
```

查看某个数据库中所有的表（查看mysql数据库中所有的表）

```sql
select * from information_schema.tables where table_schema='mysql';
```

#### 结合load\_file\(\)读取服务器文件的内容



函数LOAD\_FILE（file\_name）：读取文件并将文件内容按照字符串的格式返回。 前提条件：

* 文件的位置必须在服务器上，你必须为文件制定路径全名，而且你还必须拥有FILE特许权。 
* 文件必须可读取，文件容量必须小于max\_allowed\_packet字节。
*  若文件不存在，或因不满足上述条件而不能被读取，则函数返回值为NULL。 

1oad\_file（）用在MySQL中可以在UNOIN中充当一个字段，读取web服务器的文件

![load\_file\(\)&#x8BFB;&#x53D6;&#x6587;&#x4EF6;](../.gitbook/assets/image%20%2864%29.png)

### 多列数据拼接为一个字符串 group\_concat\(\)

```sql
select 1 union select group_concat(TABLE_NAME) from information_schema.tables where table_schema='mysql';
```

![](../.gitbook/assets/image%20%285%29.png)

## 挖掘SQL注入

### 测试注入点

| 测试字符串 | 变种 | 预期结果 |
| :--- | :--- | :--- |
| ' |  | 触发错误。如果成功，数据库将返回一个错误 |
| 1' or '1'='1 | 1'\) or \('1'='1 | 如果成功，将返回表中所有的行 |
| value' or '1'='2 | value'\) or \('1'='2 | 空条件。如果成功，将返回与原来的值相同的结果 |
| 1' and '1' = '2 | 1'\) and \('1' = '2 | 永假条件。如果成功，将不返回表中任何行 |
| 1' or 'ab'='a'+'b | 1'\) or \('ab'='a'+'b | SQL Server字符串连接。如果成功，将返回与永真条件相同的信息 |
| 1' or 'ab'='a' 'b | 1'\) or\( 'ab'='a' 'b | MySQL字符串连接。如果成功，将返回与永真条件相同的信息 |
| 1' or 'ab'='a'\|\|'b | 1'\) or \('ab'='a'\|\|'b | Oracle字符串连接。如果成功，将返回与永真条件相同的信息 |

### 数据库注释语句

| 数据库 | 注释 | 描述 |
| :--- | :--- | :--- |
| Sal server和 Oracle | -- \(double bash\) | 用于单行注释 |
|  | /\* \*/ | 多行注释 |
| Mysql | -- \(double bash\) | 用于单行注释。要求第二个dash后面跟一个空格或控制字符（如制表符、换行符等） |
|  | \# | 单行注释，但是在url中，\#号作为锚点链接，所以使用前需要URL编码 |
|  | /\*  \*/ | 用于多行注释 |



