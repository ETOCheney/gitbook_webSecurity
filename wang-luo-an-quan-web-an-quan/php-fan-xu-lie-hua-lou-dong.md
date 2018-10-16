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





