---
description: 理解Java的反射机制对理解Java的反序列化漏洞有很大帮助
---

# JAVA的反射机制

## JAVA反射的概念

> 反射（Reflection）是java的特征之一，它允许java在运行期间可以动态的获取程序自身的信息，并且可以操作类和对象的内部属性和方法。通过反射，我们可以在运行中获取程序或程序集中的每一个类型成员和成员的信息。

程序中一般的对象的类型都是在编译期就确定下来的，而java反射机制可以动态的创建对象并调用其属性和方法。这样的对象在编译期间是无法预知的，只有当程序运行时才动态加载。所以我们可以通过反射机制直接创建对象。

反射的核心就是在JVM运行时才动态加载类或调用方法/访问属性，它不需要事先（指写代码的时候或编译期间）知道运行的对象是谁。

### Java反射的功能

* **在运行时判断**任意一个对象所属的类
* **在运行时构造**任意一个类的对象
* **在运行时判断**任意一个类所具有的的成员变量和方法（通过反射甚至可以调用private方法）
* **在运行时调用**任意一个对象的方法

## 反射的主要用途

一提到反射，没有几年经验的程序员可能一时想不起来的到底能用来干什么（我便是如此）。但是我曾经看到过这样一个例子，才知道反射原来离我们并不遥远。当你在用IDE开发时，当我们的键盘在对象名字后面敲击一个.（点）时，编译器就会自动列出这个对象所拥有的属性和方法，使得开发期间十分的便捷。编译器如何知道对象所属类的属性和方法那？其实这个地方就是用到的反射，运行时（即当程序员输入代码时）动态加载需要加载的对象。

对于框架的开发人员来说，反射虽小但作用非常大，他是各种容器实现的核心。而对于一般的开发人员来说，不深入框架开发则反射用到的就会少一些。

## 反射的基本运用

> 既然反射可以用于判断任意对象的所属的类，获得Class对象，构造任意的一个对象以及调用一个对象。那么该如何去调用那？在理解了概念后我们去亲自实现一下，只用动手才能真正的了解。

### 获得Class对象

#### 1 使用Class类的forName静态方法：

```java
package HelloWorld;

public class sss {

	public static void main(String[] args) throws ClassNotFoundException {
		// TODO Auto-generated method stub
		Class<?> a = null;
		String tstring = "HelloWorld.HelloWorld";
		a = Class.forName(tstring);
		System.out.println("包名"+a.getPackage().getName()+"\t类名"+a.getName());
	}

}
```

输出：

```bash
包名HelloWorld	类名HelloWorld.HelloWorld
```

#### 2 调用某个对象的getClass\(\)方法

```java
HelloWorld b = new HelloWorld();
Class<?> bClass = b.getClass();
System.out.println("包名"+bClass.getPackage().getName()+"\t类名"+bClass.getName());
```

执行结果

```bash
包名HelloWorld	类名HelloWorld.HelloWorld
```

#### 3 直接获取某一个对象的Class

```java
Class<?> c = HelloWorld.class;
System.out.println("包名"+c.getPackage().getName()+"\t类名"+c.getName());
```

输出结果

```text
包名HelloWorld	类名HelloWorld.HelloWorld
```

### 判断是否为某个类的实例

> 一般地，我们可以用instanceof关键字来判断是否为某个类的实例。同时，我们也可以借助反射中Class对象的isInterface\(\)方法来判断。他是一个native方法。

#### 1 使用instanceof关键字

```java
HelloWorld b = new HelloWorld();
		
if (b instanceof HelloWorld) {
	System.out.println("b instanceof HelloWorld");
}
```

输出结果：

```text
b instanceof HelloWorld
```

#### 2 使用isInterface\(\)方法

```java
		HelloWorld a = new HelloWorld(); 
		HelloWorld b = new HelloWorld();
		Class<?> bClass = b.getClass();
		
		boolean isH1 = bClass.isInstance(a);
		boolean isH2 = bClass.isInstance(b);
		System.out.println(isH1+"\t"+isH2);
```

输出结果:

```text
true	true
```

### 创建实例



