---
layout : life
title : "深入理解设计模式之原型模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-15
---

##原型模式

-------------
* 原型模式是一种创建型设计模式,它通过复制一个已经存在的实例来返回新的实例,而不是新建实例.被复制的实例就是我们所称的原型,这个原型是可定制的。
* 原型模式多用于**创建复杂的或者耗时的实例**, 因为这种情况下,复制一个已经存在的实例可以使程序运行更高效,或者创建值相等,只是命名不一样的同类数据。

* 原型模式中的拷贝分为"浅拷贝"和"深拷贝":
 * **浅拷贝**: 对值类型的成员变量进行值的复制,对引用类型的成员变量只复制引用,不复制引用的对象.
 * **深拷贝**: 对值类型的成员变量进行值的复制,对引用类型的成员变量也进行引用对象的复制.

* 原型模式有两种表现形式：
 * 简单形式，别的对象在调用clone方法的时候直接new一个。
 * 登记形式，别的方法在调用clone方法的时候交给Manager去管理。

-----------

{% highlight java %}
public interface Prototype{
    /**
     * 克隆自身的方法
     * @return 一个从自身克隆出来的对象
     */
    public Object clone();
}
{% endhighlight %}
-----------
{% highlight java %}
public class ConcretePrototype implements Prototype {
    public Prototype clone(){
        //最简单的克隆，新建一个自身对象，由于没有属性就不再复制值了
        Prototype prototype = new ConcretePrototype();
        return prototype;
    }
}
{% endhighlight %}
-----------
{% highlight java %}
public class Client {
    /**
     * 持有需要使用的原型接口对象
     */
    private Prototype prototype;
    /**
     * 构造方法，传入需要使用的原型接口对象
     */
    public Client(Prototype prototype){
        this.prototype = prototype;
    }
    public void operation(Prototype example){
        //需要创建原型接口的对象
        Prototype copyPrototype = prototype.clone();
        
    }
}
{% endhighlight %}
-----------

##java中的克隆方法

--------------

* 对象的复制有一个基本问题，就是**对象通常都有对其他的对象的引用**。当使用Object类的clone()方法来复制一个对象时，此对象对其他对象的引用也同时会被复制一份
* Java语言提供的Cloneable接口只起一个作用，就是在运行时期通知Java虚拟机可以安全地在这个类上使用clone()方法。通过调用这个clone()方法可以得到一个对象的复制。由于Object类本身并不实现Cloneable接口，因此如果所考虑的类没有实现Cloneable接口时，调用clone()方法会抛出CloneNotSupportedException异常。
* 克隆满足的条件
 * clone()方法将对象复制了一份并返还给调用者。所谓**复制**的含义与clone()方法是怎么实现的。一般而言，clone()方法满足以下的描述：
 * 对任何的对象x，都有：x.clone()!=x。换言之，克隆对象与原对象不是同一个对象。
 * 对任何的对象x，都有：x.clone().getClass() == x.getClass()，换言之，克隆对象与原对象的类型一样。
 * 如果对象x的equals()方法定义其恰当的话，那么x.clone().equals(x)应当成立的。
 * 在JAVA语言的API中，凡是提供了clone()方法的类，都满足上面的这些条件。JAVA语言的设计师在设计自己的clone()方法时，也应当遵守着三个条件。一般来说，上面的三个条件中的前两个是必需的，而第三个是可选的。

---------------

##浅克隆和深克隆

---------------
* **浅克隆**
 * 　　只负责克隆按值传递的数据（比如基本数据类型、String类型），而不复制它所引用的对象，换言之，所有的对其他对象的引用都仍然指向原来的对象。
* **深克隆**
 * 除了浅度克隆要克隆的值外，还负责克隆引用类型的数据。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，深度克隆把要复制的对象所引用的对象都复制了一遍，而这种对被引用到的对象的复制叫做间接复制。
 * 深度克隆要深入到多少层，是一个不易确定的问题。在决定以深度克隆的方式复制一个对象的时候，必须决定对间接复制的对象时采取浅度克隆还是继续采用深度克隆。因此，在采取深度克隆时，需要决定多深才算深。此外，在深度克隆的过程中，很可能会出现循环引用的问题，必须小心处理。

---------------

##利用序列化实现深度克隆

----------------

* 把对象写到流里的过程是序列化(Serialization)过程；而把对象从流中读出来的过程则叫反序列化(Deserialization)过程。应当指出的是，写到流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。
* 在Java语言里深度克隆一个对象，常常可以先使对象实现Serializable接口，然后把对象（实际上只是对象的拷贝）写到一个流里（序列化），再从流里读回来（反序列化），便可以重建对象。

-----------------

{% highlight java %}
    public  Object deepClone() throws IOException, ClassNotFoundException{
        //将对象写到流里
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        //从流里读回来
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return ois.readObject();
    }
{% endhighlight %}

---------------

##浅复制

---------------


{% highlight java %}
 package com.shxz.prototype;

public class Wife {
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

{% endhighlight %}

---------------

{% highlight java %}
 package com.shxz.prototype;

public class ProtoType implements Cloneable{
	String name;
	private Wife myWife;
	
	public ProtoType(){
	}
	
	public Wife getMyWifeName() {
		return myWife;
	}

	public void setMyWifeName(Wife myWifeName) {
		this.myWife = myWifeName;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Object clone()
	{
		try {
			return  super.clone();
		} catch (CloneNotSupportedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return null;
		}
	}
	
}
{% endhighlight %}

---------------

{% highlight java %}
	public static void main(String[] args) {
		ProtoType protoType1=new ProtoType();
		Wife wife=new Wife();
		wife.setName("prototype1 wife");
		protoType1.setName("prototype1");
		protoType1.setMyWifeName(wife);
		ProtoType protoType2=(ProtoType) protoType1.clone();
		
		protoType2.setName("prototype2");
		System.out.println(protoType1.getName());
		System.out.println(protoType2.getName());
		System.out.println(protoType1.getMyWifeName());
		System.out.println(protoType2.getMyWifeName());
		protoType2.getMyWifeName().setName("protoType2 wife");
		System.out.println(protoType1.getMyWifeName().getName());
		System.out.println(protoType2.getMyWifeName().getName());
	}
	
{% endhighlight %}
##运行结果：

----------------
>prototype1
>prototype2
>com.shxz.prototype.Wife@27e6ac83
>com.shxz.prototype.Wife@27e6ac83
>protoType2 wife
>protoType2 wife

----------------

##深复制

----------------

{% highlight java %}
package com.shxz.prototype;

import java.io.Serializable;

public class Wife implements Serializable{
	//比浅复制多实现了Serializable接口
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

}

{% endhighlight %}

---------------

{% highlight java %}
 package com.shxz.prototype;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class ProtoType implements Cloneable,Serializable{
	//比浅复制多实现了Serializable接口
	String name;
	private Wife myWife;
	
	public ProtoType(){
	}
	
	public Wife getMyWifeName() {
		return myWife;
	}

	public void setMyWifeName(Wife myWifeName) {
		this.myWife = myWifeName;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Object clone()
	{
		try {
			return  super.clone();
		} catch (CloneNotSupportedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return null;
		}
	}
	 public  Object deepClone(){
	        //将对象写到流里
	        try {
				ByteArrayOutputStream bos = new ByteArrayOutputStream();
				ObjectOutputStream oos = new ObjectOutputStream(bos);
				oos.writeObject(this);
				//从流里读回来
				ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
				ObjectInputStream ois = new ObjectInputStream(bis);
				return ois.readObject();
			} catch (ClassNotFoundException e) {
			
				// TODO Auto-generated catch block
				e.printStackTrace();
				return null;
			} catch (IOException e) {
				// TODO Auto-generated catch block
				
				e.printStackTrace();
				return null;
			}
	    }
{% endhighlight %}

---------------

{% highlight java %}
	public static void main(String[] args) {
		ProtoType protoType1=new ProtoType();
		Wife wife=new Wife();
		wife.setName("prototype1 wife");
		protoType1.setName("prototype1");
		protoType1.setMyWifeName(wife);
		ProtoType protoType2=(ProtoType) protoType1.deepclone();
		
		protoType2.setName("prototype2");
		System.out.println(protoType1.getName());
		System.out.println(protoType2.getName());
		System.out.println(protoType1.getMyWifeName());
		System.out.println(protoType2.getMyWifeName());
		protoType2.getMyWifeName().setName("protoType2 wife");
		System.out.println(protoType1.getMyWifeName().getName());
		System.out.println(protoType2.getMyWifeName().getName());
	}
	
{% endhighlight %}
##运行结果：

----------------
>prototype1
>prototype2
>com.shxz.prototype.Wife@23ec4333
>com.shxz.prototype.Wife@262f6be5
>prototype1 wife
>protoType2 wife

----------------
[参考博文](http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html)