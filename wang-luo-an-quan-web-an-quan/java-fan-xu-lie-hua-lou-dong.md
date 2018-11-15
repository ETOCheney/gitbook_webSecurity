# JAVA反序列化漏洞

## JAVA 反序列化

新建一个就Java类

```java
package college;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class Student2 implements Serializable{

    private String name;  
    private int age;  

    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getAge() {  
        return age;  
    }  
    public void setAge(int age) {  
        this.age = age;  
    }  
};

public class binTest1 {

	public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
		// TODO Auto-generated method stub
		Student2 s1 = new Student2();
		s1.setName("qian");
		s1.setAge(20);
		ObjectOutputStream oo = new ObjectOutputStream(new FileOutputStream(new File("Student1.txt")));
		oo.writeObject(s1);
		oo.close();
		
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("Student1.txt")));
		Student2 s2 = (Student2)ois.readObject();
		System.out.println(s2.getName());
		ois.close();
	}

}

```

执行结果

`qian`

上面的例子就是一个student2对象s2通过反序列化得到的

## 漏洞利用演示

```java
package college;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class Student3 implements Serializable{

    private String name;  
    private int age;  

    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getAge() {  
        return age;  
    }  
    public void setAge(int age) {  
        this.age = age;  
    }  

     private void readObject(java.io.ObjectInputStream in) throws IOException,ClassNotFoundException{
    	 in.defaultReadObject();
    	 System.out.println("Unserialize");
    	 Runtime.getRuntime().exec("calc.exe");
     }

};

public class binTest2 {

	public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
		// TODO Auto-generated method stub
		Student3 s1 = new Student3();
		s1.setName("sun");
		s1.setAge(23);
		ObjectOutputStream oo = new ObjectOutputStream(new FileOutputStream(new File("student2.txt")));
		oo.writeObject(s1);
		oo.close();
		
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("student2.txt")));
		ois.readObject();
		ois.close();
	}

}

```

通过重写Student2对象的readObject方法，利用反序列化，运行时会弹出计算器

![](../.gitbook/assets/image%20%28157%29.png)



