---
layout : life
title : "深入理解设计模式之单例模式"
category : 深入理解设计模式
duoshuo: true
date : 2014-11-16
---

##单例模式

-------------

* 作为对象的创建模式，单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。
* **特点**
 * 单例类只能有一个实例。
 * 单例类必须自己创建自己的唯一实例。
 * 单例类必须给所有其他对象提供这一实例。
* 两种形式：
 * **饿汉式**：每次访问该对象先判断是否被初始化。
 * **懒汉式**：每次访问直接new一个对象返回回去。
 
-------------------------

##懒汉式

--------------------------
{% highlight java %}
package com.shxz.Singleton;

public class Singleton {
	
	private static Singleton singleton;
	private Singleton(){
	}
	public static Singleton getInstance()
	{
		if(singleton==null)
		{
			singleton=new Singleton();
			
		}
		return singleton;
	}
}

{% endhighlight %}
-----------

##饿汉式

--------------------------
{% highlight java %}
package com.shxz.Singleton;

public class Singleton2 {
	private static Singleton2 singleton=new Singleton2();
	private Singleton2(){
	}
	public static Singleton2 getInstance()
	{
		return singleton;
	}
}
{% endhighlight %}
-----------